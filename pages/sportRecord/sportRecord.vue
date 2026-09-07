<template>
	<view>
		<text>&nbsp;您已训练{{ record.length }}次</text>
		<button class="rin-btn" type="primary" size="mini" plain="true" @click="clearSportRecord">全部清除</button>
		<button class="rin-btn" :class="{ 'active': showType == 1 }" @click="toggleShow(1)">列表</button>
		<button class="rin-btn" :class="{ 'active': showType == 2 }" @click="toggleShow(2)">日历</button>
		<button class="rin-btn" :class="{ 'active': showType == 3 }" @click="toggleShow(3)">统计</button>
		<uni-list v-if="showType === 1">
			<uni-list-item v-for="(item, index) in record" clickable @click="onClick(item, index)">
				<template v-slot:header>
					<view class="slot-box">
						<i class="iconfont icon-yangwoqizuo" v-if="item.type === 'ywqz'"></i>
						<i class="iconfont icon-yaling" v-if="item.type === 'yl'"></i>
						{{ item.record }}
					</view>
				</template>
			</uni-list-item>
		</uni-list>
		<view v-if="showType === 2">
			<!-- 插入模式 日历 -->
			<uni-calendar class="uni-calendar--hook" :selected="info.selected" :showMonth="true" :lunar="info.lunar"
				:insert="info.insert" @change="change" @monthSwitch="monthSwitch" />
			<uni-list-item v-for="(item, index) in dayRecord" clickable @click="onClick(item, index)">
				<template v-slot:header>
					<view class="slot-box">
						<i class="iconfont icon-yangwoqizuo" v-if="item.type === 'ywqz'"></i>
						<i class="iconfont icon-yaling" v-if="item.type === 'yl'"></i>
						{{ item.record }}
					</view>
				</template>
			</uni-list-item>
		</view>
		<view v-if="showType === 3">
			<view class="range-tabs">
				<button class="range-arrow" @click="shiftStats(-1)">&lt;</button>
				<button class="range-btn" :class="{ 'active': statsRange === 'week' }"
					@click="changeStatsRange('week')">周</button>
				<button class="range-btn" :class="{ 'active': statsRange === 'month' }"
					@click="changeStatsRange('month')">月</button>
				<button class="range-btn" :class="{ 'active': statsRange === 'year' }"
					@click="changeStatsRange('year')">年</button>
				<button class="range-arrow" @click="shiftStats(1)">&gt;</button>
			</view>
			<view class="range-label">{{ statsPeriodLabel }}</view>
			<view :prop="chartOption" :change:prop="echarts.updateEcharts" :chart-init="chartInit"
				:change:chart-init="echarts.initEcharts" id="sport-record-chart"
				class="sport-record-chart"></view>
		</view>
	</view>
	<view>
		<uni-popup ref="alertDialog" type="dialog">
			<uni-popup-dialog :type="msgType" cancelText="取消" confirmText="确定" content="是否确定要删除该记录？" @confirm="dialogConfirm"
				@close="dialogClose"></uni-popup-dialog>
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
			chartOption: {},
			chartInit: 0,
			statsRange: 'month',
			statsCursor: '',
		}
	},
		computed: {
			statsPeriodLabel() {
				const cursor = this.getStatsCursor();
				if (this.statsRange === 'week') {
					const day = cursor.getDay() || 7;
					const weekStart = new Date(cursor);
					weekStart.setDate(cursor.getDate() - day + 1);
					const weekEnd = new Date(weekStart);
					weekEnd.setDate(weekStart.getDate() + 6);
					return this.formatDisplayDate(weekStart) + ' - ' + this.formatDisplayDate(weekEnd);
				}
				if (this.statsRange === 'year') {
					return cursor.getFullYear() + '年01月01日 - ' + cursor.getFullYear() + '年12月31日';
				}
				const monthStart = new Date(cursor.getFullYear(), cursor.getMonth(), 1);
				const monthEnd = new Date(cursor.getFullYear(), cursor.getMonth() + 1, 0);
				return this.formatDisplayDate(monthStart) + ' - ' + this.formatDisplayDate(monthEnd);
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
			this.updateChartOption();
		},
		updateChartOption() {
			const dailyRecords = {};
			this.record.forEach(item => {
				const date = item.date || (item.record && item.record.substring(0, 10));
				if (!date) return;
				if (!dailyRecords[date]) {
					dailyRecords[date] = { ywqz: 0, yl: 0 };
				}
				dailyRecords[date][item.type === 'yl' ? 'yl' : 'ywqz'] += 1;
			});
			const now = this.getStatsCursor();
			let dates = [];
			let dateFormatter;
			if (this.statsRange === 'week') {
				const weekStart = new Date(now);
				const day = weekStart.getDay() || 7;
				weekStart.setDate(weekStart.getDate() - day + 1);
				for (let index = 0; index < 7; index++) {
					const date = new Date(weekStart);
					date.setDate(weekStart.getDate() + index);
					dates.push(this.formatDate(date));
				}
				dateFormatter = date => date.substring(5);
			} else if (this.statsRange === 'year') {
				for (let month = 1; month <= 12; month++) {
					dates.push(now.getFullYear() + '-' + (month < 10 ? '0' : '') + month);
				}
				dateFormatter = date => date.substring(5, 7) + '月';
			} else {
				const daysInMonth = new Date(now.getFullYear(), now.getMonth() + 1, 0).getDate();
				for (let day = 1; day <= daysInMonth; day++) {
					const date = new Date(now.getFullYear(), now.getMonth(), day);
					dates.push(this.formatDate(date));
				}
				dateFormatter = date => date.substring(8) + '日';
			}

			const getRecordCount = (date, type) => {
				if (this.statsRange === 'year') {
					return Object.keys(dailyRecords).reduce((total, recordDate) => {
						return total + (recordDate.substring(0, 7) === date ? dailyRecords[recordDate][type] : 0);
					}, 0);
				}
				return dailyRecords[date] ? dailyRecords[date][type] : 0;
			};

			this.chartOption = {
				color: ['#007aff', '#f39c12'],
				tooltip: { trigger: 'axis' },
				legend: { data: ['仰卧起坐', '哑铃'] },
				grid: { left: 45, right: 20, bottom: 45, top: 45 },
				xAxis: { type: 'category', data: dates.map(dateFormatter) },
				yAxis: { type: 'value', minInterval: 1, name: '次数' },
				series: [
					{ name: '仰卧起坐', type: 'line', smooth: true, data: dates.map(date => getRecordCount(date, 'ywqz')) },
					{ name: '哑铃', type: 'line', smooth: true, data: dates.map(date => getRecordCount(date, 'yl')) }
				]
			};
		},
		formatDate(date) {
				const month = date.getMonth() + 1;
				const day = date.getDate();
				return date.getFullYear() + '-' + (month < 10 ? '0' : '') + month + '-' + (day < 10 ? '0' : '') + day;
		},
			formatDisplayDate(date) {
				return date.getFullYear() + '年' + (date.getMonth() + 1) + '月' + date.getDate() + '日';
			},
		getStatsCursor() {
			return this.statsCursor ? new Date(this.statsCursor + 'T00:00:00') : new Date();
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
				success: function () {
					// console.log('success');
					uni.showToast({
						title: '删除运动记录成功',
						icon: 'none',
					});
				}
			});
			this.setCalendarData();
			this.updateChartOption();
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
				success: function () {
					// console.log('success');
					uni.showToast({
						title: '删除运动记录成功',
						icon: 'none',
					});
				}
			});
			this.setCalendarData();
			this.updateChartOption();
		},
		toggleShow(type) {
			this.showType = type;
			if (type === 3) {
				this.updateChartOption();
				this.chartInit++;
			}
		},
		changeStatsRange(range) {
			this.statsRange = range;
			this.updateChartOption();
		},
		shiftStats(direction) {
			const cursor = this.getStatsCursor();
			if (this.statsRange === 'week') {
				cursor.setDate(cursor.getDate() + direction * 7);
			} else if (this.statsRange === 'year') {
				cursor.setFullYear(cursor.getFullYear() + direction);
			} else {
				cursor.setMonth(cursor.getMonth() + direction);
			}
			this.statsCursor = this.formatDate(cursor);
			this.updateChartOption();
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

<script module="echarts" lang="renderjs">
	let myChart;
	let initTimer;
	let latestOption = {};

	export default {
		unmounted() {
			if (initTimer) {
				clearTimeout(initTimer);
				initTimer = null;
			}
			if (myChart) {
				myChart.dispose();
				myChart = null;
			}
		},
		methods: {
			initEcharts(newValue) {
				if (!window.echarts) {
					const script = document.createElement('script');
					script.src = 'static/echarts.js';
					script.onload = () => this.initEcharts(1);
					document.head.appendChild(script);
					return;
				}
				this.createChart(0);
			},
			createChart(retryCount) {
				const chartElement = document.getElementById('sport-record-chart');
				if (!chartElement || !chartElement.offsetWidth || !chartElement.offsetHeight) {
					if (retryCount < 20) {
						initTimer = setTimeout(() => this.createChart(retryCount + 1), 50);
					}
					return;
				}
				if (myChart) myChart.dispose();
				myChart = window.echarts.init(chartElement);
				myChart.setOption(latestOption || this.chartOption || {});
				myChart.resize();
			},
			updateEcharts(newValue) {
				latestOption = newValue || {};
				if (myChart) myChart.setOption(latestOption, true);
			}
		}
	}
</script>

<style>
.rin-btn {
	margin-left: 10px;
	display: inline-block;
	line-height: 2.2;
	font-size: 13px;
	padding: 0 1em;
	color: #007aff;
	border: 1px solid #007aff;
}

.active {
	color: #fff;
	background-color: #007aff;
}

.sport-record-chart {
	width: 100%;
	height: 420px;
}

.range-tabs {
	padding: 12px 10px 0;
	text-align: center;
}

.range-btn {
	display: inline-block;
	box-sizing: border-box;
	height: 30px;
	min-height: 30px;
	margin: 0;
	padding: 0 10px;
	line-height: 28px;
	font-size: 13px;
	vertical-align: middle;
	color: #007aff;
	background-color: #fff;
	border: 1px solid #007aff;
	border-radius: 0;
}

.range-btn + .range-btn {
	margin-left: -1px;
}

.range-btn.active {
	color: #fff;
	background-color: #007aff;
}

.range-label {
	padding-top: 8px;
	color: #666;
	font-size: 13px;
	text-align: center;
}

.range-arrow {
	display: inline-block;
	box-sizing: border-box;
	width: 30px;
	height: 30px;
	min-height: 30px;
	margin: 0 6px;
	padding: 0;
	line-height: 28px;
	font-size: 18px;
	vertical-align: middle;
	color: #007aff;
	background-color: #fff;
	border: 1px solid #007aff;
	border-radius: 0;
}
</style>