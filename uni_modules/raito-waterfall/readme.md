# 瀑布流布局解决方案raito-waterfall，已在小程序、H5、vue2项目实践过，app或vue3需要改造一下，思路差不多

## 简述
raito-waterfall是本人实践过来的瀑布流布局方案，简单易用、兼容vue2、小程序、H5等。

**关于列表key：** 列表唯一key值是必填的，否则会出现不可预测的表现，同时也是高性能的必要条件，所以您的数据中一定要有唯一值，比如id。

**数据更新：** 您可以随意地更新data数据，渲染列表会在您每次更新后重新排列。

**关于渲染项：** 您可以自定义渲染项的样式，raito-waterfall-item只是提供一种最简单的布局方式。但无论您如何定义渲染项，都需要emit渲染项的实际高度。如果您的渲染项有图片，也需要在图片加载完后重新emit高度，可参考raito-waterfall-item对于动态高度的解决方式。

**关于图片：** 图片大小建议限制在100k以内，过大的图片会影响渲染速度。

**实现思路：** 瀑布流的布局思路基本是将渲染项放在高度最小的一列，所以需要获取渲染项的高度后才能正确渲染。本组件是直接将数据渲染出骨架屏，不是挨个渲染，然后当监听到渲染项的高度变化后将列表重新排列。

## 使用方式
### 1. 导入插件
使用HbuilderX导入到项目的uni_modules文件夹中

### 2. 示例
完整示例请下载示例项目，然后使用HbuilderX运行。
```html
<raito-waterfall 
	keyName="id"
	:data="data" 
	:count="2" 
	:gutter="10" 
	:lrPading="10" 
	:skeletonHeight="240" 
	:animation="true"
>
</raito-waterfall>
```

### 3. raito-waterfall属性说明

| 参数         | 说明         | 类型        | 是否必填     | 可选值        | 默认值 |
| ----------- | ----------- | ----------- | ----------- | ----------- | ----------- |
| keyName     | 列表key值    | String      | 是          | -            | -       |
| data        | 数据列表     | Array        | 是         | -             | []       |
| count       | 渲染列数（2-5列）| Number    | 否         | 2-5            | 2       |
| gutter       | 渲染项间隔（单位px）| Number    | 否         | -            | 10       |
| lrPading       | 渲染列表左右padding（单位px）| Number    | 否         | -            | 10       |
| skeletonHeight       | 骨架屏高度（单位px）| Number    | 否         | -            | 0       |
| animation       | 是否开启动画效果| Boolearn    | 否         | -            | true       |


## 扫码体验
<br/>
<img src="https://raito-liao.github.io/raito-waterfall/static/h5-demo-qrcode.png" width="200px"><br/>
H5扫码体验(国内可能会慢一些)

---
**如何您喜欢该插件，记得收藏、好评哦，比心哟！**

作者：Raito Liao  

邮箱：raito.liao@outlook.com