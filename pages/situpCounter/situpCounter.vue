<template>
	<view>
		<view class="uni-common-mt">
			<text class="uni-subtitle">本页面会播放音频,请注意音量</text>
			<uni-forms ref="form" :modelValue="formData" :rules="rules" label-position="top" label-width="100%"
				validateTrigger="bind">
				<uni-forms-item required label="每组次数（整数）：" name="value1">
					<uni-easyinput class="uni-mt-5" trim="all" v-model="formData.value1" placeholder="每组次数（整数）"
						@input="input"></uni-easyinput>
				</uni-forms-item>
				<uni-forms-item required label="每次时间间隔（秒，可以有小数）：" name="value2">
					<uni-easyinput class="uni-mt-5" trim="all" v-model="formData.value2" placeholder="每次时间间隔（秒，可以有小数）"
						@input="input"></uni-easyinput>
				</uni-forms-item>
				<uni-forms-item required label="组数（整数）：" name="value3">
					<uni-easyinput class="uni-mt-5" trim="all" v-model="formData.value3" placeholder="组数（整数）"
						@input="input"></uni-easyinput>
				</uni-forms-item>
				<uni-forms-item required label="每组时间间隔（秒，可以有小数，建议8以上)：" name="value4">
					<uni-easyinput class="uni-mt-5" trim="all" v-model="formData.value4"
						placeholder="每组时间间隔（秒，可以有小数，建议8以上)" @input="input"></uni-easyinput>
				</uni-forms-item>
			</uni-forms>

			<view class="uni-form-item uni-column">
				<view class="button-group">
					<button class="rin-btn" type="primary" size="default" v-if="!onStart" @click="startBtn">开始</button>
					<button class="rin-btn" type="primary" size="default" v-if="onStart && !onStop"
						@click="pause">暂停</button>
					<button class="rin-btn" type="primary" size="default" v-if="onStart && onStop"
						@click="resume">继续</button>
					<button class="rin-btn" type="primary" size="default" v-if="onStart" @click="stopClick">停止</button>
					<button class="rin-btn" type="primary" size="default" @click="clearAll">清除</button>
				</view>
				<view class="button-group">
					<button class="rin-btn" type="primary" size="mini" plain="true" @click="saveConfig">保存配置</button>
					<button class="rin-btn" type="primary" size="mini" plain="true" @click="removeConfig">清除配置</button>
					<button class="rin-btn" type="primary" size="mini" plain="true"
						@click="saveSportRecord">保存记录</button>
				</view>
				<view class="button-group">
					<text class="tipmsg-1">(配置保存在本地缓存)</text>
				</view>
			</view>
			<text class="uni-subtitle">
				<text>您已完成：{{groupNum}}组，{{number}}次 </text>
			</text>
			<br>
			<text class="uni-subtitle" v-if="tipMessage2">{{tipMessage2}}</text>
			<br v-if="tipMessage2">
			<text class="uni-subtitle">
				<text>用时 {{tipMessage3}}</text>
			</text>
			<br>
			<text class="uni-subtitle">{{tipMessage}}</text>
			<br v-if="tipMessage">

			<!-- 训练进度条 -->
			<view class="progress-wrap" v-if="onStart">
				<!-- 整体训练进度 -->
				<view class="progress-row">
					<text class="progress-label">总进度</text>
					<view class="progress-track">
						<view class="progress-fill progress-fill--total" :style="{ width: progressOverallPct + '%' }">
						</view>
					</view>
					<text class="progress-pct">{{ groupNum }}/{{ formData.value3 }}组</text>
				</view>
				<view class="progress-row">
					<text class="progress-label">组进度</text>
					<view class="progress-track">
						<view class="progress-fill progress-fill--set" :style="{ width: progress1Pct + '%' }"></view>
					</view>
					<text class="progress-pct">{{ Math.round(progress1Pct) }}%</text>
				</view>
				<view class="progress-row">
					<text class="progress-label">次进度</text>
					<view class="progress-track">
						<view class="progress-fill progress-fill--rep" :style="{ width: progress2Pct + '%' }"></view>
					</view>
					<text class="progress-pct">{{ Math.round(progress2Pct) }}%</text>
				</view>
			</view>

		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				currentSide: 1, // 1左 2右

				formData: {
					value1: '50', // 每组次数
					value2: '2.4', // 每次间隔
					value3: '4', // 组数
					value4: '180', // 每组间隔
					value5: 0, // 是否区分左右
					value7: '3', // 左右切换时间
				},
				rules: {
					value1: {
						rules: [{
								required: true,
								errorMessage: '请输入每组次数'
							},
							{
								format: 'number',
								errorMessage: '请输入数字'
							}
						]
					},
					value2: {
						rules: [{
								required: true,
								errorMessage: '请输入每次间隔'
							},
							{
								format: 'number',
								errorMessage: '请输入数字'
							}
						]
					},
					value3: {
						rules: [{
								required: true,
								errorMessage: '请输入组数'
							},
							{
								format: 'number',
								errorMessage: '请输入数字'
							}
						]
					},
					value4: {
						rules: [{
								required: true,
								errorMessage: '请输入每组间隔'
							},
							{
								format: 'number',
								errorMessage: '请输入数字'
							}
						]
					},
				},

				// tts

				timer: null, // 组内个数计时 Interval
				timer2: null, // 组歇 timeout
				interval3: null, // 已消耗时间计时
				startTime: null, // 开始锻炼时间戳（Date对象）
				number: 0, // 组内已完成锻炼个数
				groupNum: 0, // 已完成锻炼组数
				tipMessage: '', // 提示信息
				tipMessage2: '', // 提示信息（预计耗时）
				tipMessage3: '00:00:00', // 已锻炼时长
				useSeconds: 0, // 锻炼已用秒数（暂停时保留）
				_timerBaseTime: 0, // 当前计时段的起始时间戳（用于消除漂移）
				_lastTickTime: 0, // 上一次训练 tick 触发的时间戳（用于息屏补偿）
				_groupAnnouncing: false, // start() 已调用但播报音频尚未结束
				_totalDoneReps: 0, // 已完成的总次数（只增不减，用于总进度条）
				intervalCountdown: null, // 组歇/切换倒计时 interval（需统一管理）
				onStop: false, // 当前是否正在暂停
				onStart: false, // 当前是否正在进行锻炼
				audioObjAndroid: null, // android端audio对象
				audioObjH5: null, // H5端audio对象
				sideList: [{
						text: '是',
						value: 1
					},
					{
						text: '否',
						value: 0
					}
				],
				// 进度条计时器
				echartsInterval1: null,
				echartsInterval2: null,
				echartsValue1: 0, // 组内总进度（0 ~ v1*v2）
				echartsValue2: 0, // 单次进度（0 ~ v2）

				audioContext: null,
			}
		},
		onReady() {
			this.getConfig();
			// H5：注册息屏恢复监听
			// #ifdef H5
			this._onVisibilityChange = () => {
				if (document.visibilityState === 'visible' && this.onStart && !this.onStop) {
					this.recoverFromSleep();
				}
			};
			document.addEventListener('visibilitychange', this._onVisibilityChange);
			// #endif
		},
		// App 端：屏幕亮起 / 从后台切回时触发
		onShow() {
			if (this.onStart && !this.onStop) {
				this.recoverFromSleep();
			}
		},
		onHide() {
			// 仅取消屏幕常亮；训练计时器继续运行，等屏幕恢复时补偿
			this.setKeepScreenOn(false);
		},
		onUnload() {
			// #ifdef H5
			if (this._onVisibilityChange) {
				document.removeEventListener('visibilitychange', this._onVisibilityChange);
			}
			// #endif
			this.stopCounter(false);
			this.setKeepScreenOn(false);
		},
		computed: {
			// 组内总进度百分比（外条）：echartsValue1 / (v1*v2) * 100
			progress1Pct() {
				const max = Number(this.formData.value1) * Number(this.formData.value2);
				if (!max) return 0;
				return Math.min(100, (this.echartsValue1 / max) * 100);
			},
			// 单次进度百分比（内条）：echartsValue2 / v2 * 100
			progress2Pct() {
				const max = Number(this.formData.value2);
				if (!max) return 0;
				return Math.min(100, (this.echartsValue2 / max) * 100);
			},

			/**
			 * 整体训练进度百分比
			 * 纯按已完成次数 / 总次数计算，不与时间或内圈进度关联
			 */
			progressOverallPct() {
				if (!this.onStart) return 0;
				const v1 = Number(this.formData.value1);
				const v3 = Number(this.formData.value3);
				const hasLR = this.formData.value5 === 1;
				const totalReps = v3 * v1 * (hasLR ? 2 : 1);
				if (totalReps <= 0) return 0;
				return Math.min(100, (this._totalDoneReps / totalReps) * 100);
			},
		},
		methods: {
			input(e) {
				// console.log('输入内容：', e);
			},


			/**
			 * 对表单进行校验，校验通过调用回调函数
			 * @param {Function} handler
			 */
			validateForm(handler) {
				this.$refs.form.validate().then(() => {
					handler();
				}).catch(err => {
					console.log('表单校验失败：', err);
					uni.showToast({
						title: '配置校验失败，' + err[0].errorMessage,
						icon: 'none',
					});
				});
			},

			/**
			 * 设置屏幕常亮
			 * @param {Boolean} tag
			 */
			setKeepScreenOn(tag) {
				// #ifdef APP-PLUS
				uni.setKeepScreenOn({
					keepScreenOn: tag
				});
				plus.device.setWakelock(tag);
				// #endif
			},

			/**
			 * 计算预计耗时
			 *
			 * 实际训练时序（普通模式）：
			 *   开始播报(~1.5s)
			 *   × v3 组：[第X组播报(~2s) → v1次 × v2秒 → 缓冲(0.8s)]
			 *   + (v3-1) × v4 秒组歇（最后一组无组歇）
			 *   + 完成播报(~5s)
			 *
			 * 左右模式每组额外增加：
			 *   左侧播报差值(+0.5s) + 左侧完成播报(~2s) + v7秒切换 + 右侧播报(~2.5s) + 右侧完成播报(~1.5s)
			 *   即每组多：2×v1×v2 + v7 + 6.5s（左右各跑一遍）
			 */
			countTimeSpend() {
				const v1 = Number(this.formData.value1); // 每组次数
				const v2 = Number(this.formData.value2); // 每次间隔(秒)
				const v3 = Number(this.formData.value3); // 组数
				const v4 = Number(this.formData.value4); // 每组间隔(秒)
				const hasLR = this.formData.value5 === 1;
				const v7 = Number(this.formData.value7); // 左右切换时间(秒)

				// 音频时长常量（估算值，单位秒）
				const T_START = 1.5; // "开始" 播报
				const T_SET = 2.0; // "第X组" 播报（普通模式）
				const T_SET_LR = 2.5; // "第X组左/右侧" 播报（左右模式，含侧别播报）
				const T_LR_DONE = 2.0; // "左侧完成，请换到右侧" 播报
				const T_R_DONE = 1.5; // "右侧完成" 播报
				const T_BUF = 0.8; // 每组结束到进入组歇的缓冲
				const T_END = 5.0; // 完成所有组合播报

				let exerciseTime; // 纯训练时间（所有组加起来）
				if (hasLR) {
					// 每个逻辑组 = 左侧跑一遍 + 切换 + 右侧跑一遍
					const perSet = T_SET_LR + v1 * v2 + T_LR_DONE + v7 + T_SET_LR + v1 * v2 + T_R_DONE + T_BUF;
					exerciseTime = v3 * perSet;
				} else {
					const perSet = T_SET + v1 * v2 + T_BUF;
					exerciseTime = v3 * perSet;
				}

				const restTime = (v3 - 1) * v4; // 最后一组无组歇
				const totalTime = T_START + exerciseTime + restTime + T_END;

				// 格式化为 m分s秒 或直接秒数
				const totalSec = Math.round(totalTime);
				const minutes = Math.floor(totalSec / 60);
				const seconds = totalSec % 60;
				const timeStr = minutes > 0 ? `${minutes}分${seconds}秒` : `${seconds}秒`;

				// 分项说明，方便核对
				const exerciseSec = Math.round(exerciseTime);
				const restSec = Math.round(restTime);
				this.tipMessage2 = `预计耗时 ${timeStr}` +
					`（训练${exerciseSec}秒 + 组歇${restSec}秒）`;
			},

			/**
			 * 开始按钮点击
			 */
			startBtn() {
				this.validateForm(() => {
					this.stopCounter(true);
					this.setKeepScreenOn(true);
					this.onStart = true;
					const musicList = this.formData.value5 === 1 ? ['czcks'] : ['kaishi'];
					this.playAudioList(musicList, () => {
						this.start();
					});
					this.countTimeSpend();
					this.startTimer3(true);
				});
			},

			/**
			 * 开始一组训练
			 */
			start() {
				this.tipMessage = '正在训练';
				if (this.timer) {
					clearInterval(this.timer);
					this.timer = null;
				}
				this.number = 0;
				this.groupNum++;

				let musicList = ['di', ...this.splitNumber(this.groupNum), 'zu'];
				if (this.formData.value5 === 1) {
					musicList.push(this.currentSide === 1 ? 'zuoce' : 'youce');
				}
				this._groupAnnouncing = true;
				this.playAudioList(musicList, () => {
					this._groupAnnouncing = false;
					this.startCounterIntenval(true);
				});
			},

			/**
			 * 用时统计
			 * @param {Boolean} startTag - true: 重新开始计时，false: 从暂停处继续
			 */
			startTimer3(startTag) {
				if (startTag) {
					this.startTime = new Date();
					this.useSeconds = 0;
				}
				if (this.interval3) {
					clearInterval(this.interval3);
					this.interval3 = null;
				}
				// 用真实时间差计算，消除 setInterval 漂移
				const baseSeconds = this.useSeconds;
				this._timerBaseTime = Date.now();
				this.interval3 = setInterval(() => {
					this.useSeconds = baseSeconds + Math.floor((Date.now() - this._timerBaseTime) / 1000);
					const h = Math.floor(this.useSeconds / 3600);
					const m = Math.floor((this.useSeconds % 3600) / 60);
					const s = this.useSeconds % 60;
					this.tipMessage3 = [h, m, s].map(n => this.addZero(n)).join(':');
				}, 1000);
			},

			/**
			 * 息屏恢复补偿：计算息屏期间漏掉的训练 tick 并批量补齐
			 * 同时重新对齐 timer 到下一个应触发的时刻，避免短时间内双重触发
			 */
			recoverFromSleep() {
				if (!this.onStart || !this.timer || !this._lastTickTime) return;
				const now = Date.now();
				const intervalMs = Number(this.formData.value2) * 1000;
				const elapsed = now - this._lastTickTime;

				// 未超过一个间隔，说明没有漏 tick，无需补偿
				if (elapsed < intervalMs) return;

				// 计算漏掉的次数（减1是因为补偿后会立即重启 timer 触发下一次）
				const missed = Math.floor(elapsed / intervalMs) - 1;
				console.log(`息屏补偿：漏掉 ${missed + 1} 次，补偿 ${missed} 次`);

				// 补偿漏掉的 tick（跳过最后一个，由重启的 timer 正常触发）
				for (let i = 0; i < missed; i++) {
					this._lastTickTime = Date.now();
					this.startCounterIntenvalItem();
					if (!this.onStart || !this.timer) return; // 提前结束（如恰好完成一组）
				}

				// 重启 timer：对齐到下一个应触发的时刻
				if (this.timer) {
					clearInterval(this.timer);
					const remaining = intervalMs - (elapsed % intervalMs);
					setTimeout(() => {
						if (!this.onStart || this.onStop) return;
						this._lastTickTime = Date.now();
						this.startCounterIntenvalItem();
						// 重新建立正常节拍
						this.timer = setInterval(() => {
							this._lastTickTime = Date.now();
							this.startCounterIntenvalItem();
						}, intervalMs);
					}, remaining);
				}
			},

			/**
			 * 组内个数计时及播放
			 * @param {Boolean} tag - true: 从头开始，false: 继续
			 */
			startCounterIntenval(tag) {
				if (!this.onStart) return;
				this.startEchartsTimer1(tag);
				this._lastTickTime = Date.now();
				this.startCounterIntenvalItem();
				this.timer = setInterval(() => {
					this._lastTickTime = Date.now();
					this.startCounterIntenvalItem();
				}, Number(this.formData.value2) * 1000);
			},

			startCounterIntenvalItem() {
				this.number++;
				if (this.number <= Number(this.formData.value1)) {
					this._totalDoneReps++;
					this.startEchartsTimer2();
					this.playMusic();
				}
				// 完成一组，进入组歇
				if (this.number > Number(this.formData.value1)) {
					if (this.timer) {
						clearInterval(this.timer);
						this.timer = null;
					}
					this.number = 0;
					this.stopEchartsTimer1(true);
					this.stopEchartsTimer2(true);
					// 区分左右：左侧数完再数右侧
					if (this.formData.value5 === 1 && this.currentSide === 1) {
						this.currentSide = 2;
						this.groupNum--;
						const musicList = ['zuoce', 'wancheng', 'qhdyc'];
						this.tipMessage = '左侧完成，休息' + this.formData.value7 + '秒';
						this.playAudioList(musicList, () => {
							let second = Number(this.formData.value7);
							if (this.intervalCountdown) {
								clearInterval(this.intervalCountdown);
							}
							this.intervalCountdown = setInterval(() => {
								if (second === 0) {
									clearInterval(this.intervalCountdown);
									this.intervalCountdown = null;
									return;
								}
								second--;
								this.tipMessage = '左侧完成，休息' + second + '秒';
							}, 1000);
							this.timer2 = setTimeout(() => {
								this.start();
							}, Number(this.formData.value7) * 1000);
						});
						return;
					} else if (this.formData.value5 === 1 && this.currentSide === 2) {
						this.currentSide = 1;
						this.playAudioList(['youce', 'wancheng'], () => {
							setTimeout(() => {
								this.groupWait();
							}, 800);
						});
						return;
					} else {
						// 不区分左右
						setTimeout(() => {
							this.groupWait();
						}, 800);
					}
				}
			},

			/**
			 * 每组完成：进行组歇播报或全部完成播报
			 */
			groupWait() {
				if (this.groupNum < Number(this.formData.value3)) {
					this.tipMessage = '组歇中，休息' + this.formData.value4 + '秒';
					let musicList = ['nywc', 'di',
						...this.splitNumber(this.groupNum),
						'zu', 'zuxie',
						...this.splitNumber(this.formData.value4),
						'miao'
					];
					let second = Number(this.formData.value4);
					if (this.intervalCountdown) {
						clearInterval(this.intervalCountdown);
					}
					this.intervalCountdown = setInterval(() => {
						if (second === 0) {
							clearInterval(this.intervalCountdown);
							this.intervalCountdown = null;
							return;
						}
						second--;
						this.tipMessage = '组歇中，休息' + second + '秒';
					}, 1000);
					this.playAudioList(musicList);
					this.timer2 = setTimeout(() => {
						this.start();
					}, Number(this.formData.value4) * 1000);
				} else {
					this.tipMessage = '完成所有组合';
					let musicList = ['wcsyzh', 'nywc', 'gong', ...this.splitNumber(this.groupNum), 'zu'];
					if (this.formData.value5 === 1) musicList.push('zuoyou', 'ge');
					const total = Number(this.formData.value1) * Number(this.groupNum);
					musicList.push(...this.splitNumber(total), 'ci');
					this.playAudioList(musicList);
					this.saveSportRecord();
					// 重置训练状态（不清零组数/次数，保留完成数据展示）
					this.stopCounter(false);
				}
			},

			/**
			 * 暂停按钮点击
			 */
			pause() {
				this.tipMessage = '暂停, 组歇中暂停再继续可跳过组歇';
				this.playAudioList(['zanting']);
				if (this.timer) {
					clearInterval(this.timer);
					this.timer = null;
				}
				this.stopEchartsTimer1(false);
				this.stopEchartsTimer2(true);
				if (this.timer2) {
					clearTimeout(this.timer2);
					this.timer2 = null;
				}
				if (this.intervalCountdown) {
					clearInterval(this.intervalCountdown);
					this.intervalCountdown = null;
				}
				if (this.interval3) {
					clearInterval(this.interval3);
					this.interval3 = null;
				}
				this.onStop = true;
			},

			/**
			 * 继续按钮点击
			 */
			resume() {
				this.validateForm(() => {
					this.tipMessage = '继续';
					this.playAudioList(['jixu'], () => {
						this.startTimer3(false);
						if (this.number !== 0) {
							// 组中暂停
							this.startCounterIntenval(false);
						} else {
							// 组歇暂停，直接开始下一组
							this.start();
						}
					});
					this.onStop = false;
				});
			},

			/**
			 * 停止按钮点击
			 */
			stopClick() {
				this.tipMessage = '停止';
				setTimeout(() => {
					this.playAudioList(['tingzhi']);
				}, 0);
				this.stopCounter(false);
				this.clearAudioObj();
			},

			stopCounter(resetNumTag) {
				if (resetNumTag) {
					this.number = 0;
					this.groupNum = 0;
					this.currentSide = 1;
					this._totalDoneReps = 0;
				}
				this.startTime = null;
				this.onStart = false;
				this._groupAnnouncing = false;
				if (this.timer) {
					clearInterval(this.timer);
					this.timer = null;
				}
				if (this.timer2) {
					clearTimeout(this.timer2);
					this.timer2 = null;
				}
				if (this.intervalCountdown) {
					clearInterval(this.intervalCountdown);
					this.intervalCountdown = null;
				}
				if (this.interval3) {
					clearInterval(this.interval3);
					this.interval3 = null;
				}
				this.stopEchartsTimer1(false);
				this.stopEchartsTimer2(true);
				this.setKeepScreenOn(false);
			},

			/**
			 * 清除所有计时器和已完成数量
			 */
			clearAll() {
				setTimeout(() => {
					this.playAudioList(['qingchu']);
				}, 0);
				this.stopCounter(true);
				this.clearAudioObj();
			},

			clearAudioObj() {
				// #ifdef APP-PLUS
				if (plus.os.name === 'Android' && this.audioObjAndroid) {
					this.audioObjAndroid.pause();
					this.audioObjAndroid.stop();
					this.audioObjAndroid.destroy();
					this.audioObjAndroid = null;
				}
				// #endif
				// #ifdef H5
				if (this.audioObjH5) {
					this.audioObjH5.pause();
					this.audioObjH5 = null;
				}
				// #endif
			},

			/**
			 * 组内计数个数播放
			 */
			playMusic() {
				let num = this.number;
				if (this.number % 10 !== 0) {
					num = this.number % 10;
				}
				if (num <= 100) {
					this.playAudioList(['num_' + num]);
				} else if (num < 1000) {
					// 大于一百的整十数，如 110 --> 一百 一十
					if (this.number % 10 !== 0) {
						const musicList = [];
						musicList.push('num_' + parseInt(num / 100) + '00');
						const tens = parseInt((num % 100) / 10);
						musicList.push(tens === 1 ? 'yishi' : 'num_' + tens + '0');
						this.playAudioList(musicList);
					}
				} else {
					this.tipMessage = '超过一千';
					this.playAudioList(['cgyqzbzc']);
					this.stop();
				}
			},

			/**
			 * 分解数字为音频列表，如 123 → [一百, 二十, 三]，支持 1000 以内
			 * @param {Number} num
			 * @returns {Array}
			 */
			splitNumber(num) {
				if (num >= 1000) return ['cgyqzbzc']; // 修复: >= 1000
				const list = [];
				const thousands = parseInt(num / 1000);
				if (thousands > 0) list.push('num_' + thousands + '000');
				const hundreds = parseInt((num % 1000) / 100);
				if (hundreds > 0) list.push('num_' + hundreds + '00');
				const tens = parseInt((num % 100) / 10);
				if (tens > 0) list.push('num_' + tens + '0');
				const ones = parseInt(num % 10);
				if (ones > 0) list.push('num_' + ones);
				return list;
			},

			/**
			 * 依次播放传入的音频列表
			 * @param {Array} musicList
			 * @param {Function} handler 播放完成后回调
			 */
			playAudioList(musicList, handler) {
				// #ifdef APP-PLUS
				if (plus.os.name === 'Android') {
					this.playAudioListAndroid(musicList, handler);
				}
				// #endif
				// #ifdef H5
				this.playAudioListH5(musicList, handler);
				// #endif
			},

			/**
			 * H5 端用原生 Audio 播放（合并多段音频后播放）
			 * @param {Array} musicList
			 * @param {Function} handler
			 */
			playAudioListH5(musicList, handler) {
				try {
					if (!this.audioContext) {
						this.audioContext = new AudioContext();
					}
					// 重建 audio 对象，避免旧实例残留
					if (this.audioObjH5) {
						this.audioObjH5.pause();
					}
					this.audioObjH5 = new Audio();

					const play = (src) => {
						this.audioObjH5.src = src;
						this.audioObjH5.load();
						this.audioObjH5.play();
						// 使用 { once: true } 防止监听器重复堆叠
						this.audioObjH5.addEventListener('ended', () => {
							this.audioObjH5.pause();
							this.audioObjH5.src = null;
							if (handler) handler();
						}, {
							once: true
						});
					};

					if (musicList.length > 1) {
						const srcList = musicList.map(m => '/static/mp3/' + m + '.mp3');
						this.concatAudioObj(srcList).then(src => play(src));
					} else {
						play('/static/mp3/' + musicList[0] + '.mp3');
					}
				} catch (e) {
					console.log('audioObjH5 catchError', e);
					if (this.audioObjH5) {
						this.audioObjH5.pause();
						this.audioObjH5 = null;
					}
				}
			},

			/**
			 * Android 端用 uni.createInnerAudioContext 顺序播放音频列表
			 * @param {Array} musicList
			 * @param {Function} handler
			 */
			/**
			 * Android 端顺序播放音频列表
			 * 每个文件独立创建 InnerAudioContext，彻底避免 onEnded/onCanplay 监听器跨调用堆积，
			 * 解决训练完成提示在 Android 上不播放的问题。
			 * @param {Array} musicList
			 * @param {Function} handler 全部播完后回调
			 */
			playAudioListAndroid(musicList, handler) {
				// 中断并销毁当前正在播放的音频
				if (this.audioObjAndroid) {
					try {
						this.audioObjAndroid.stop();
						this.audioObjAndroid.destroy();
					} catch (e) {}
					this.audioObjAndroid = null;
				}

				const playNext = (index) => {
					// 全部播完，触发回调
					if (index >= musicList.length) {
						this.audioObjAndroid = null;
						if (handler) handler();
						return;
					}
					let audio;
					try {
						audio = uni.createInnerAudioContext();
						this.audioObjAndroid = audio;
						audio.src = '/static/mp3/' + musicList[index] + '.mp3';

						audio.onEnded(() => {
							audio.destroy();
							if (this.audioObjAndroid === audio) this.audioObjAndroid = null;
							playNext(index + 1);
						});

						// 出错时跳过当前文件，继续播下一个
						audio.onError((e) => {
							console.log('audioObjAndroid onError', JSON.stringify(e), audio.src);
							audio.destroy();
							if (this.audioObjAndroid === audio) this.audioObjAndroid = null;
							playNext(index + 1);
						});

						audio.onCanplay(() => {
							audio.play();
							audio.offCanplay();
						});
					} catch (e) {
						console.log('audioObjAndroid catchError', e);
						if (audio) {
							try {
								audio.destroy();
							} catch (e2) {}
						}
						if (this.audioObjAndroid === audio) this.audioObjAndroid = null;
						playNext(index + 1);
					}
				};

				playNext(0);
			},

			async concatAudioObj(audioSrcArray) {
				const bufferList = await Promise.all(audioSrcArray.map(src => this.getAudioBuffer(src)));
				const concatBuffer = this.concatAudio(bufferList);
				// 注意：createObjectURL 生成的 URL 需在不再使用时调用 URL.revokeObjectURL 释放
				return URL.createObjectURL(this.bufferToWave(concatBuffer, concatBuffer.length));
			},

			/**
			 * 获取音频 Buffer（已加 reject 处理）
			 * @param {String} src
			 * @returns {Promise<AudioBuffer>}
			 */
			async getAudioBuffer(src) {
				const response = await fetch(src);
				const arrayBuffer = await response.arrayBuffer();
				return this.audioContext.decodeAudioData(arrayBuffer);
			},

			// 拼接多个 AudioBuffer
			concatAudio(bufferList) {
				const maxChannels = Math.max(...bufferList.map(b => b.numberOfChannels));
				const totalLength = bufferList.reduce((sum, b) => sum + b.length, 0);
				const output = this.audioContext.createBuffer(maxChannels, totalLength, bufferList[0].sampleRate);
				let offset = 0;
				bufferList.forEach(buf => {
					for (let ch = 0; ch < buf.numberOfChannels; ch++) {
						output.getChannelData(ch).set(buf.getChannelData(ch), offset);
					}
					offset += buf.length;
				});
				return output;
			},

			// AudioBuffer 转 WAV Blob
			bufferToWave(abuffer, len) {
				const numOfChan = abuffer.numberOfChannels;
				const length = len * numOfChan * 2 + 44;
				const buffer = new ArrayBuffer(length);
				const view = new DataView(buffer);
				const channels = [];
				let i, sample, offset = 0,
					pos = 0;

				const setUint16 = (data) => {
					view.setUint16(pos, data, true);
					pos += 2;
				};
				const setUint32 = (data) => {
					view.setUint32(pos, data, true);
					pos += 4;
				};

				setUint32(0x46464952); // "RIFF"
				setUint32(length - 8);
				setUint32(0x45564157); // "WAVE"
				setUint32(0x20746d66); // "fmt "
				setUint32(16);
				setUint16(1); // PCM
				setUint16(numOfChan);
				setUint32(abuffer.sampleRate);
				setUint32(abuffer.sampleRate * 2 * numOfChan);
				setUint16(numOfChan * 2);
				setUint16(16);
				setUint32(0x61746164); // "data"
				setUint32(length - pos - 4);

				for (i = 0; i < abuffer.numberOfChannels; i++) channels.push(abuffer.getChannelData(i));
				while (pos < length) {
					for (i = 0; i < numOfChan; i++) {
						sample = Math.max(-1, Math.min(1, channels[i][offset]));
						sample = (0.5 + sample < 0 ? sample * 32768 : sample * 32767) | 0;
						view.setInt16(pos, sample, true);
						pos += 2;
					}
					offset++;
				}
				return new Blob([buffer], {
					type: 'audio/wav'
				});
			},


			/**
			 * 组内训练个数计时器（进度条：外条 - 组进度）
			 * @param {Boolean} clearNum - 是否重置进度
			 */
			startEchartsTimer1(clearNum) {
				if (clearNum) this.echartsValue1 = 0;
				if (this.echartsInterval1) {
					clearInterval(this.echartsInterval1);
					this.echartsInterval1 = null;
				}
				const v1 = Number(this.formData.value1);
				const v2 = Number(this.formData.value2);
				this.echartsInterval1 = setInterval(() => {
					this.echartsValue1 += v1 / 100;
					if (this.echartsValue1 >= v1 * v2) this.echartsValue1 = 0;
				}, v1 * 1000 / 100);
			},

			/**
			 * 停止外圈计时器
			 * @param {Boolean} tag - true: 等当前圈走完再清除；false: 立即清除
			 */
			stopEchartsTimer1(tag) {
				if (tag) {
					// 修复：用 || 代替 && 防止内部轮询永不退出（interval 为 null 但 value 非 0 的情况）
					const guard = setTimeout(() => {
						clearInterval(inner);
						this._clearEchartsInterval1();
					}, 3000);
					const inner = setInterval(() => {
						if (this.echartsValue1 === 0 || !this.echartsInterval1) {
							this._clearEchartsInterval1();
							clearInterval(inner);
							clearTimeout(guard);
						}
					}, 2);
				} else {
					this._clearEchartsInterval1();
				}
			},

			_clearEchartsInterval1() {
				if (this.echartsInterval1) {
					clearInterval(this.echartsInterval1);
					this.echartsInterval1 = null;
				}
			},

			/**
			 * 每次完成进度计时器（进度条：内条 - 次进度）
			 */
			startEchartsTimer2() {
				if (this.echartsInterval2) {
					clearInterval(this.echartsInterval2);
					this.echartsInterval2 = null;
				}
				this.echartsValue2 = 0;
				this.echartsInterval2 = setInterval(() => {
					const next = Number((this.echartsValue2 + 0.02).toFixed(2));
					if (next >= Number(this.formData.value2)) {
						this.echartsValue2 = 0;
					} else {
						this.echartsValue2 = next;
					}
				}, 20);
			},

			/**
			 * 停止内圈计时器
			 * @param {Boolean} tag - true: 等当前圈走完再清除；false: 立即清除
			 */
			stopEchartsTimer2(tag) {
				if (tag) {
					const guard = setTimeout(() => {
						clearInterval(inner);
						this._clearEchartsInterval2();
					}, 3000);
					const inner = setInterval(() => {
						if (this.echartsValue2 === 0 || !this.echartsInterval2) {
							this._clearEchartsInterval2();
							clearInterval(inner);
							clearTimeout(guard);
						}
					}, 2);
				} else {
					this._clearEchartsInterval2();
				}
			},

			_clearEchartsInterval2() {
				if (this.echartsInterval2) {
					clearInterval(this.echartsInterval2);
					this.echartsInterval2 = null;
				}
			},

			/**
			 * 保存配置到本地缓存
			 */
			saveConfig() {
				const config = JSON.stringify({
					value1: this.formData.value1 || '',
					value2: this.formData.value2 || '',
					value3: this.formData.value3 || '',
					value4: this.formData.value4 || '',
				});
				uni.setStorage({
					key: 'rinson_toolbox_config_2',
					data: config,
					success() {
						uni.showToast({
							title: '保存配置成功',
							icon: 'none'
						});
					}
				});
			},

			/**
			 * 读取本地缓存配置
			 */
			getConfig() {
				try {
					const value = uni.getStorageSync('rinson_toolbox_config_2');
					if (value) {
						const config = JSON.parse(value);
						['value1', 'value2', 'value3', 'value4'].forEach(key => {
							if (config[key]) this.formData[key] = config[key];
						});
					}
				} catch (e) {
					console.error('读取配置失败', e);
				}
			},

			/**
			 * 清除本地缓存配置
			 */
			removeConfig() {
				uni.removeStorage({
					key: 'rinson_toolbox_config_2',
					success() {
						uni.showToast({
							title: '清除配置成功',
							icon: 'none'
						});
					}
				});
			},

			/**
			 * 保存运动记录（去除两个分支的重复代码）
			 */
			saveSportRecord() {
				const now = this.startTime || new Date();
				const total = Number(this.formData.value1) * Number(this.groupNum);
				const newEntry = {
					record: this.timeFormat(now) + ' 仰卧起坐：' + this.groupNum + '组，' + total + '次',
					type: 'ywqz',
					date: this.dateFormat(now),
				};
				try {
					const stored = uni.getStorageSync('rinson_toolbox_sportRecord');
					const list = stored ? JSON.parse(stored) : [];
					list.unshift(newEntry);
					uni.setStorage({
						key: 'rinson_toolbox_sportRecord',
						data: JSON.stringify(list),
						success() {
							uni.showToast({
								title: '保存运动记录成功',
								icon: 'none'
							});
						}
					});
				} catch (e) {
					console.error('保存运动记录失败', e);
				}
			},

			/**
			 * 格式化为 yyyy-MM-dd HH:mm:ss
			 */
			timeFormat(time) {
				const pad = (n) => this.addZero(n);
				return `${time.getFullYear()}-${pad(time.getMonth() + 1)}-${pad(time.getDate())} ` +
					`${pad(time.getHours())}:${pad(time.getMinutes())}:${pad(time.getSeconds())}`;
			},

			/**
			 * 格式化为 yyyy-MM-dd（复用 timeFormat，不重复逻辑）
			 */
			dateFormat(time) {
				return this.timeFormat(time).slice(0, 10);
			},

			/**
			 * 数字补零（重命名自 addZreo，修正拼写）
			 */
			addZero(n) {
				return String(n).padStart(2, '0');
			},
		}
	}
</script>


<style scoped lang="scss">
	.uni-common-mt {
		padding: 20rpx 40rpx;
	}

	.form-item {
		margin-top: 0rpx;
	}

	.uni-forms-item {
		margin-bottom: 20rpx;
	}

	.button-group {
		margin-top: 0rpx;
	}

	.rin-btn {
		display: inline-block;
		margin: 0 10rpx;
	}

	.progress-wrap {
		margin-top: 24rpx;
		padding: 0 10rpx;
	}

	.progress-row {
		display: flex;
		align-items: center;
		margin-bottom: 20rpx;
	}

	.progress-label {
		width: 80rpx;
		font-size: 26rpx;
		color: #555;
		flex-shrink: 0;
	}

	.progress-track {
		flex: 1;
		height: 20rpx;
		background: #e8e8e8;
		border-radius: 10rpx;
		overflow: hidden;
		margin: 0 16rpx;
	}

	.progress-fill {
		height: 100%;
		border-radius: 10rpx;
		transition: width 0.1s linear;
	}

	.progress-fill--total {
		background: linear-gradient(90deg, #36c090, #0fa870);
	}

	.progress-fill--set {
		background: linear-gradient(90deg, #4f8eff, #2255cc);
	}

	.progress-fill--rep {
		background: linear-gradient(90deg, #5aabff, #1e88e5);
	}

	.progress-pct {
		width: 70rpx;
		font-size: 24rpx;
		color: #333;
		text-align: right;
		flex-shrink: 0;
	}

	.tipmsg-1 {
		font-size: 14px;
	}
</style>