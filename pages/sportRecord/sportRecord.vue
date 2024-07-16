<template>
	<view>
		<button class="rin-btn" type="primary" size="mini" plain="true" @click="clearSportRecord">全部清除</button>
		<text>&nbsp;您已训练{{record.length}}次</text>
		<uni-list>
			<uni-list-item v-for="(item,index) in record"  clickable @click="onClick(item, index)">
				<template v-slot:header>
					<view class="slot-box">
						<i class="iconfont icon-yangwoqizuo" v-if="item.type === 'ywqz'"></i>
						<i class="iconfont icon-yaling" v-if="item.type === 'yl'"></i>
						{{item.record}}
					</view>
				</template>
			</uni-list-item>
		</uni-list>
	</view>
	<view>
		<uni-popup ref="alertDialog" type="dialog">
			<uni-popup-dialog :type="msgType" cancelText="关闭" confirmText="同意" content="是否确定要删除该记录？"
				@confirm="dialogConfirm" @close="dialogClose"></uni-popup-dialog>
		</uni-popup>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				record: [],
				msgType: 'warn',
				selectItem: null,
			}
		},
		onReady() {
			this.getConfig();
		},
		methods: {
			getConfig() {
				const value = uni.getStorageSync('rinson_toolbox_sportRecord');
				if (value) {
					this.record = JSON.parse(value);
				}
			},
			onClick(item, index) {
				// console.log('执行click事件', item)
				this.selectItem = {
					item,
					index
				};
				this.$refs.alertDialog.open();
			},
			dialogConfirm() {
				// console.log('点击确认')
				// this.messageText = `点击确认了 ${this.msgType} 窗口`
				// this.$refs.message.open()
				this.record = this.record.filter(item => item !== this.selectItem.item);
				uni.setStorage({
					key: 'rinson_toolbox_sportRecord',
					data: JSON.stringify(this.record),
					success: function() {
						// console.log('success');
						uni.showToast({
							title: '删除运动记录成功',
							icon: 'none',
						});
					}
				});
			},
			dialogClose() {
				this.selectItem = null;
				// console.log('点击关闭')
			},
			clearSportRecord() {
				this.record = [];
				uni.setStorage({
					key: 'rinson_toolbox_sportRecord',
					data: JSON.stringify(this.record),
					success: function() {
						// console.log('success');
						uni.showToast({
							title: '删除运动记录成功',
							icon: 'none',
						});
					}
				});
			}
		}
	}
</script>

<style>

</style>