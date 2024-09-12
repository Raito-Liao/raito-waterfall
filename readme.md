# 在uniapp插件市场上一直找不到合适的瀑布流组件，于是自己动手写了一个，并在项目中实践出来的瀑布流解决方案

###  一、我对于瀑布流布局组件的要求
1. 要有骨架屏，能提前将基本结构展示给用户；
2. 不要挨个渲染，否则如果有图片的话加载将会很慢；
3. 可以实现多列瀑布流；
4. 绝对不能出现错位渲染，很多瀑布流组件都出现这种情况，特别在页面跳转或Tabs切换下。
5. 渲染过程中不能有卡顿影响用户体验。
6. 应用于小程序、H5、App等

###  二、插件地址
完整代码请点击链接后下载示例项目：<br/>
[https://ext.dcloud.net.cn/plugin?id=20167](https://ext.dcloud.net.cn/plugin?id=20167)

### 三、组件文档
[https://github.com/Raito-Liao/raito-waterfall/blob/master/uni_modules/raito-waterfall/readme.md](https://github.com/Raito-Liao/raito-waterfall/blob/master/uni_modules/raito-waterfall/readme.md)

### 四、实现讲解
1. 计算渲染项的位置
原理都差不多，在获得渲染项的宽高后，将渲染项定位在最小高度的那一列的下面。
```javascript
// raito-waterfall部分代码
export default {
		methods: {
			generateRenderList() {
				const renderList = []
				const {
					columnCount,
					columnWidth,
					skeletonHeight,
					gutter,
					lrPading,
					keyName,
					cache
				} = this
				const columnHeights = new Array(columnCount).fill(0);
			
				if (!keyName) {
					throw new Error('keyName is required!')
				}
				
				this.data.forEach((item, index) => {
					const itemKey = item[keyName]
					
					if (!cache[itemKey]) {
						cache[itemKey] = {
							top: 0,
							left: 0,
							width: columnWidth,	// item宽度
							height: skeletonHeight // 骨架屏高度
						}
					}
					
					if (index < columnCount) {
						cache[itemKey].top = 0
						cache[itemKey].left = lrPading + index * (columnWidth + gutter)
						columnHeights[index] += cache[itemKey].height
					} else {
						const minHeight = Math.min(...columnHeights)
						const minIndex = columnHeights.indexOf(minHeight);
						cache[itemKey].top = minHeight + gutter
						cache[itemKey].left = lrPading + minIndex * (columnWidth + gutter)
						columnHeights[minIndex] += cache[itemKey].height + gutter
					}
			
					renderList.push({
						...item,
						_layoutRect: cache[itemKey],
						_renderKey: itemKey
					})
				})
			
				const maxHeight = Math.max(...columnHeights)
				
				return {
					renderList,
					maxHeight
				}
			},
			startRender() {
				const {
					maxHeight,
					renderList
				} = this.generateRenderList()
				
				if (!renderList.length) {
					return
				}
				this.renderList = renderList
				this.waterfallBoxHeight = maxHeight
			},
		}
}
```

2. 如何避免错位渲染？<br/>
这种情况发生于页面跳转或Tabs切换，导致获取不到元素的boundingClientRect，也就获取不到元素的高度，于是会导致计算错误。
```javascript
function getRect(selector, context) {
		return new Promise((resolve, reject) => {
			const query = uni.createSelectorQuery().in(context);
			query
				.select(selector)
				.boundingClientRect(rect => {
					rect ? resolve(rect) : reject('Not Found');
				})
				.exec();
		});
}
```
可以这样解决：当createSelectorQuery获取不到boundingClientRect时，使用createIntersectionObserver来监听元素，这样同样可以获取得到boundingClientRect。当重新回到页面或Tabs切换回来时，intersectionObserver会被触发。
```javascript
// raito-waterfall-item部分代码
export default {
		mounted() {
			this.getExtraBoxWH().finally(() => {
				/* 可能由于页面跳转、元素隐藏导致获取不到宽度和高度,于是通过监听元素来重新获取高度 */
				if (!this.contentRect.extraBoxWidth || !this.contentRect.extraBoxHeight) {
					this.startObserver();
				}
			});
		},
		beforeDestroy() {
			if (this.intersectionObserver) {
				this.intersectionObserver.disconnect();
				this.intersectionObserver = null;
			}
		},
		methods: {
			startObserver() {
				this.intersectionObserver = uni.createIntersectionObserver(this, {
					thresholds: [0, 1],
					initialRatio: 0,
					observeAll: false
				});
				this.intersectionObserver.relativeToViewport();
				this.intersectionObserver.observe('.content-box__extra', res => {
					const { width, height } = res.boundingClientRect;
					this.contentRect.extraBoxWidth = width;
					this.contentRect.extraBoxHeight = height;
					this.intersectionObserver.disconnect();
					this.intersectionObserver = null;
				});
			},
			// 获取extraBox的宽高
			getExtraBoxWH() {
				return getRect('.content-box__extra', this).then(rect => {
					if (rect) {
						this.contentRect.extraBoxWidth = rect.width;
						this.contentRect.extraBoxHeight = rect.height;
					}
				});
			},
		}
}
```

3. 防止数据频繁变化和渲染项高度变化导致卡顿，需要对其节流
```javascript
import { isArraysEqual, throttle } from '../../util.js'
export default {
		created() {
			this.throttleRender = throttle(this.startRender.bind(this), 100) // 防止频繁调用
			this.handleDataChange = throttle(this.handleDataChange.bind(this), 100)	// 防止频繁调用
			this.$watch('data', this.handleDataChange, {
				deep: true,
				immediate: true
			})
		},
		methods: {
			onHeightChange(item, height) {
				const itemKey = item._renderKey
				this.cache[itemKey].height = height	
				this.throttleRender()
			},
			handleDataChange(newData, oldData) {
				if (isArraysEqual(newData, oldData)) {
					return
				}
				this.startRender()
			},
		}
}
```

### 五、扫码预览
<br/>
<img src="https://raito-liao.github.io/raito-waterfall/static/h5-demo-qrcode.png" width="200px"><br/>
H5扫码体验(国内可能会慢一些)