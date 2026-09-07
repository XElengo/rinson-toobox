<template>
	<view>
		<text>&nbsp;您已训练{{record.length}}次</text>
		<button class="rin-btn" type="primary" size="mini" plain="true" @click="clearSportRecord">全部清除</button>
		<button class="rin-btn" type="primary" size="mini" plain="true"
			@click="toggleShow">{{showType === 1 ? '日历' : '列表'}}</button>

		<view v-if="showType === 2">
			<!-- 插入模式 日历 -->
			<uni-calendar class="uni-calendar--hook" :selected="info.selected" :showMonth="true" :lunar="info.lunar"
				:insert="info.insert" @change="change" @monthSwitch="monthSwitch" />
			<uni-list-item v-for="(item,index) in dayRecord" clickable @click="onClick(item, index)">
				<template v-slot:header>
					<view class="slot-box">
						<i class="iconfont icon-yangwoqizuo" v-if="item.type === 'ywqz'"></i>
						<i class="iconfont icon-yaling" v-if="item.type === 'yl'"></i>
						{{item.record}}
					</view>
				</template>
			</uni-list-item>
		</view>
		<uni-list v-if="showType === 1">
			<uni-list-item v-for="(item,index) in record" clickable @click="onClick(item, index)">
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
			<uni-popup-dialog :type="msgType" cancelText="取消" confirmText="确定" content="是否确定要删除该记录？"
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
				showType: 1,
				info: {
					lunar: true,
					insert: true,
					selected: []
				},
				dayRecord: [],
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
					this.setCalendarData();
				}
			},
			setCalendarData() {
				this.info.selected = [];
				this.record.forEach(item => {
					if (!item.date && item.record) {
						item.date = item.record.substring(0, 10);
					}
					if (item.date) {
						let exitItem = this.info.selected.find(item2 => item.date === item2.date);
						if (exitItem && exitItem.type !== item.type) {
							exitItem.info = '哑/仰';
						} else {
							this.info.selected = [...this.info.selected, {
								date: item.date,
								type: item.type,
								info: item.type === 'yl' ? '哑铃' : '仰卧'
							}];
						}

					}
				})
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
				this.setCalendarData();
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
				this.setCalendarData();
			},
			toggleShow() {
				if (this.showType === 1) {
					this.showType = 2;
				} else {
					this.showType = 1;
				}

			},
			change(e) {
				// console.log('change 返回:', e)
				this.dayRecord = this.record.filter(item => item.date == e.fulldate);
				
			},
			monthSwitch(e) {
				// console.log('monthSwitchs 返回:', e)
			}
		}
	}
</script>

<style>
	.rin-btn {
		margin-left: 10px;
	}
</style>