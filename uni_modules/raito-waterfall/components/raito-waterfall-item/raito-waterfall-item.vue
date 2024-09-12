<template>
	<!-- waterfall-item样式根据实际情况变化，但逻辑没变 -->
	<view class="waterfall-item-box">
		<view class="skeleton-box" v-if="!hideSkeleton">
			<view class="skeletons">
				<view class="skeleton__item" style="width: 100%; height: 120px; border-radius: 5px 5px 0px 0px"></view>
				<view class="skeleton__item"
					style="width: 50%; height: 18px; margin-top: 15px; border-radius: 0rpx 5px 5px 0px"></view>
				<view class="skeleton__item"
					style="width: 80%; height: 18px; margin-top: 15px; border-radius: 0rpx 5px 5px 0px"></view>
				<view class="skeleton__item"
					style="width: 50%; height: 18px; margin-top: 15px; border-radius: 0rpx 5px 5px 0px"></view>
			</view>
		</view>

		<view class="content-box" :class="{ loading: !hideSkeleton }">
			<view class="content-box__image">
				<image :src="imageUrl" mode="widthFix" style="width: 100%; border-radius: 5px 5px 0px 0px" lazy-load
					@load="onImageLoad" @error="onImageError"></image>
			</view>
			<view class="content-box__extra">
				<view class="content-box__title">{{ itemObject.title }}</view>
				<view class="content-box__subtitle">{{ itemObject.subtitle }}</view>
			</view>

		</view>
	</view>
</template>

<script>
	import { getRect } from '../../util.js'

	export default {
		name: "RaitoWaterfallItem",
		props: {
			itemObject: {
				type: Object,
				default () {
					return {}
				}
			}
		},
		data() {
			return {
				hideSkeleton: false, // 是否隐藏骨架屏
				imageUrl: '', // 图片地址
				contentRect: {
					extraBoxWidth: 0, // extraBox宽度
					extraBoxHeight: 0, // extraBox高度
					imageRatio: 0 // 图片宽高比
				}
			}
		},
		created() {
			this.imageUrl = this.itemObject.imageUrl
			let unwatch = this.$watch(
				'contentRect',
				() => {
					const {
						extraBoxWidth,
						extraBoxHeight,
						imageRatio
					} = this.contentRect;

					const { hideSkeleton } = this;

					if (!hideSkeleton && extraBoxWidth && extraBoxHeight && imageRatio) {
						this.emitHeight();
						this.hideSkeleton = true;
						unwatch();
						unwatch = null;
					}
				}, 
				{ deep: true }
			);
		},
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
			// 图片加载成功
			onImageLoad(e) {
				const { width, height } = e.detail;
				this.contentRect.imageRatio = width / height;
			},
			// 图片加载失败
			onImageError() {
				/* 加载失败后使用默认图片（这里展示时用的是网络图片，可用本地图片替代，图片比例根据实际调整）*/
				this.contentRect.imageRatio = 4 / 3;
				this.imageUrl = 'https://fakeimg.pl/400x300';
			},
			emitHeight() {
				const {
					extraBoxWidth,
					extraBoxHeight,
					imageRatio
				} = this.contentRect;
				let height = extraBoxHeight + extraBoxWidth / imageRatio;
				this.$emit('heightChange', height);
			}
		}

	}
</script>

<style scoped lang="scss">
	.waterfall-item-box {
		width: 100%;
		box-shadow: 0px 0px 10px 0px rgba(0, 0, 0, 0.1);
		border-radius: 5px;
		background: #fff;
		position: relative;
	}

	.skeleton-box {
		// padding: 0 0 10px 0;
		height: 240px;
		box-sizing: border-box;

		.skeletons {
			.skeleton__item {
				/* #ifdef APP-NVUE */
				background-color: #f1f2f4;
				/* #endif */
				/* #ifndef APP-NVUE */
				background: linear-gradient(90deg, #f1f2f4 25%, #e6e6e6 37%, #f1f2f4 50%);
				background-size: 400% 100%;
				/* #endif */
				animation: skeleton 1.8s ease infinite;
			}
		}
	}

	@keyframes skeleton {
		0% {
			background-position: 100% 50%;
		}

		100% {
			background-position: 0 50%;
		}
	}

	.content-box {
		position: static;

		&.loading {
			position: absolute;
			width: 100%;
			top: 0;
			left: 0;
			opacity: 0;
			pointer-events: none;
			z-index: -1;
		}

		.content-box__image {
			position: relative;
			overflow: hidden;
			font-size: 0; // 防止图片白边
		}

		.content-box__extra {
			padding: 8px 10px 10px;

			.content-box__title {
				font-size: 16px;
				font-weight: bold;
				color: #000
			}

			.content-box__subtitle {
				font-size: 14px;
				color: #999;
				margin-top: 10px;
			}
		}
	}
</style>