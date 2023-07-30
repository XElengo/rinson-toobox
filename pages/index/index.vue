<template>
	<view class="content">
		<view class="text-area">
			<div class="nav-item" :prop="is_show" @click="toggle(1)" :change:prop="renderswipe.toggle">
				<i class="iconfont icon-yaling"></i>
				<div>哑铃</div>
				<div>计数器</div>
			</div>
			<div class="nav-item" :prop="is_show" @click="toggle(2)" @touchend="renderswipe.toggle">
				<i class="iconfont icon-yangwoqizuo"></i>
				<div>仰卧起坐</div>
				<div>计数器</div>
			</div>
			<div class="nav-item" :prop="is_show" @click="toggle(3)" @touchend="renderswipe.toggle">
				<i class="iconfont icon-zhangdan_o"></i>
				<div>账单记录</div>
			</div>
			<div class="nav-item" :prop="is_show" @click="toggle(4)" @touchend="renderswipe.toggle">
				<i class="iconfont icon-zhanghaomima"></i>
				<div>账号密码</div>
				<div>管理</div>
			</div>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: 'main',
				is_show: '1',
			}
		},
		onLoad() {

		},
		methods: {
			toggle(tag) {
				// if(alert) {
				// 	alert(tag);
				// }
				this.is_show = tag;
				if(tag === 1) {
					uni.navigateTo({
						url: '/pages/dumbbellCounter/dumbbellCounter'
					});
				}
				if(tag === 2) {
					uni.navigateTo({
						url: '/pages/situpCounter/situpCounter'
					});
				}
				if(tag === 3) {
					uni.navigateTo({
						url: '/pages/billingRecord/billingRecord'
					});
				}
				if(tag === 4) {
					uni.navigateTo({
						url: '/pages/accountManager/accountManager'
					});
				}
			}

		}
	}
</script>

<script module="renderswipe" lang="renderjs">
	export default {
		mounted(e, ins, owner) {

		},
		methods: {
			toggle(tag) {
				// console.log( JSON.stringify(tag) );
				// window.alert(tag.currentTarget.id);
				 // window.alert(tag);
			},
		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-flow: row wrap;
		align-content: flex-start;
		padding: 30rpx;
		width: 100%;
		height: 100%;
	}

	.text-area {
		display: flex;
		flex-wrap: wrap;
		width: 100%;
		justify-content: flex-start;
	}

	.nav-item {
		box-sizing: border-box;
		flex: 0 0 1;
		height: 260rpx;
		width: calc((100% - 180rpx) / 2);
		padding: 40rpx 20rpx;
		background-color: rgba(22, 145, 255, 0.15);
		border: 1rpx solid rgba(24, 144, 255, 0.5);
		text-align: center;
		vertical-align: middle;
		border-radius: 10rpx;
		font-size: 30rpx;
		margin: 0 30rpx 60rpx 30rpx;

		&:hover,
		&:active {
			background-color: rgba(22, 145, 255, 0.3);
		}

		.iconfont {
			display: inline-block;
			font-size: 40px;
			margin-bottom: 30rpx;
		}
	}
</style>