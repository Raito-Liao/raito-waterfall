<template>
	<view class="waterfall-box" :style="{ height: waterfallBoxHeight + 'px'}">
		<block v-for="(item, index) in renderList" :key="item._renderKey">
			<view class="waterfall-box__item" 
				:class="{ animation: animation }" 
				:style="{ 
				top: item._layoutRect.top + 'px', 
				left: item._layoutRect.left + 'px',
				width: columnWidth + 'px'
			}">
				<raito-waterfall-item :itemObject="item" @heightChange="onHeightChange(item, $event)" />
			</view>
		</block>
	</view>
</template>
<script>
	import RaitoWaterfallItem from '../raito-waterfall-item/raito-waterfall-item.vue'
	import { isArraysEqual, throttle } from '../../util.js'

	export default {
		name: "RaitoWaterfall",
		components: {
			RaitoWaterfallItem
		},
		// #ifdef MP-WEIXIN
		options: {
			virtualHost: true
		},
		// #endif
		props: {
			// 列表唯一key值（必填）
			keyName: String,
			// 实际数据（必填）
			data: {
				type: Array,
				default () {
					return []
				}
			},
			// 渲染列数，2-5列
			count: {
				type: Number,
				default: 2
			},
			// 渲染项间隙，单位px
			gutter: {
				type: Number,
				default: 10
			},
			// 渲染列表左右pading，单位px
			lrPading: {
				type: Number,
				default: 10
			},
			// 骨架屏高度，单位px
			skeletonHeight: {
				type: Number,
				default: 0
			},
			// 是否开启动画效果
			animation: {
				type: Boolean,
				default: true
			}
		},
		data() {
			return {
				// 总高度
				waterfallBoxHeight: 0,
				// 列宽
				columnWidth: 0,
				// 渲染列表
				renderList: [],
			}
		},
		created() {
			this.columnCount = this.count > 5 ? 5 : this.count < 2 ? 2 : Math.floor(this.count)
			this.columnWidth = (uni.upx2px(750) - this.lrPading * 2 - (this.columnCount - 1) * this.gutter) / this.columnCount
			this.throttleRender = throttle(this.startRender.bind(this), 100) // 防止频繁调用
			this.handleDataChange = throttle(this.handleDataChange.bind(this), 100)	// 防止频繁调用
			this.cache = {}
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
							width: columnWidth,
							height: skeletonHeight
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
</script>
<style scoped lang="scss">
	.waterfall-box {
		position: relative;
		width: 750rpx;

		.waterfall-box__item {
			position: absolute;

			&.animation {
				transition: all 0.3s linear;
			}
		}
	}
</style>