<template>
	<view class="page">
		<raito-waterfall :data="data" :count="2" :gutter="10" :lrPading="10" :skeletonHeight="240" keyName="id"
			:animation="true"></raito-waterfall>
	</view>
</template>

<script>
	let autoId = 1

	export default {
		data() {
			return {
				data: []
			}
		},
		onReachBottom() {
			// 限制一下数据频繁加载，onReachBottom有时会连续触发2次
			this.loadData()
		},
		onLoad() {
			this.loadData()
		},
		methods: {
			loadData() {
				/* 注意：图片尽量压缩到100k以内，这里为了展示方便使用fakeimg */
				const newData = [{
						id: autoId++,
						imageUrl: `https://fakeimg.pl/400x500`,
						title: "这里是一条标题",
						subtitle: "这里是一条子标题；这里是一条子标题；这里是一条子标题；这里是一条子标题"
					},
					{
						id: autoId++,
						imageUrl: `https://fakeimg.pl/300x400`,
						title: "这里是一条标题",
						subtitle: "这里是一条子标题；这里是一条子标题；这里是一条子标题；这里是一条子标题"
					},
					{
						id: autoId++,
						imageUrl: `https://fakeimg.pl/300x500`,
						title: "这里是一条标题",
						subtitle: "这里是一条子标题；这里是一条子标题；这里是一条子标题；这里是一条子标题"
					},
					{
						id: autoId++,
						imageUrl: `https://fakeimg.pl/400x450`,
						title: "这里是一条标题",
						subtitle: "这里是一条子标题；这里是一条子标题；这里是一条子标题；这里是一条子标题"
					},
					{
						id: autoId++,
						imageUrl: `https://fakeimg.pl/300x400`,
						title: "这里是一条标题",
						subtitle: "这里是一条子标题；这里是一条子标题；这里是一条子标题；这里是一条子标题"
					},
					{
						id: autoId++,
						imageUrl: `https://fakeimg.pl/350x500`,
						title: "这里是一条标题",
						subtitle: "这里是一条子标题；这里是一条子标题；这里是一条子标题；这里是一条子标题"
					},
					{
						id: autoId++,
						imageUrl: `https://fakeimg.pl/500x400`,
						title: "这里是一条标题",
						subtitle: "这里是一条子标题；这里是一条子标题；这里是一条子标题；这里是一条子标题"
					},
					{
						id: autoId++,
						imageUrl: `https://fakeimg.pl/600x500`,
						title: "这里是一条标题",
						subtitle: "这里是一条子标题；这里是一条子标题；"
					},
					{
						id: autoId++,
						imageUrl: `https://fakeimg.pl/350x450`,
						title: "这里是一条标题",
						subtitle: "这里是一条子标题；这里是一条子标题；这里是一条子标题；这里是一条子标题"
					},
					{
						id: autoId++,
						imageUrl: `https://fakeimg.pl/400x450`,
						title: "这里是一条标题",
						subtitle: "这里是一条子标题；这里是一条子标题；这里是一条子标题；这里是一条子标题；这里是一条子标题；这里是一条子标题"
					},
					{
						id: autoId++,
						imageUrl: `https://fakeimg.pl/500x500`,
						title: "这里是一条标题",
						subtitle: "这里是一条子标题；这里是一条子标题；这里是一条子标题；这里是一条子标题；这里是一条子标题；这里是一条子标题；这里是一条子标题；这里是一条子标题；这里是一条子标题；这里是一条子标题；"
					}
				]

				setTimeout(() => {
					this.data = this.data.concat(newData)
				})
			}
		}
	}
</script>

<style>
.page {
	padding-top: 15px;
	background: #f5f5f5;
}
</style>