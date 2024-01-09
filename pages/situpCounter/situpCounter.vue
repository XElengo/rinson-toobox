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
				<!-- <view class="form-item">
					<text class="uni-subtitle">播放方式：</text>
					<uni-data-checkbox mode="tag" v-model="value6" :localdata="voiceType"></uni-data-checkbox>
				</view> -->
				<!-- default -->
				<view class="button-group">
					<button class="rin-btn" type="primary" size="default" v-if="!onStart" @click="startBtn">开始</button>
					<button class="rin-btn" type="primary" size="default" v-if="onStart && !onStop"
						@click="pause">暂停</button>
					<button class="rin-btn" type="primary" size="default" v-if="onStart && onStop"
						@click="resume">继续</button>
					<button class="rin-btn" type="primary" size="default" v-if="onStart" @click="stopClick">停止</button>
					<button class="rin-btn" type="primary" size="default" @click="clearAll">清除</button>
					<!-- <button type="default" size="default" @click="countTimeSpend">计算耗时</button> -->
				</view>
				<view class="button-group">
					<button class="rin-btn" type="primary" size="mini" plain="true" @click="saveConfig">保存配置</button>
					<button class="rin-btn" type="primary" size="mini" plain="true" @click="removeConfig">清除配置</button>
					<br>
					<text class="tipmsg-1">(配置保存在本地缓存)</text>
					<!-- <button class="rin-btn" type="primary" size="default" @click="playMusic3">测试</button> -->
				</view>
			</view>
			<text class="uni-subtitle">
				<text>您已完成：{{groupNum}}组，{{number}}次 </text>
			</text>
			<br>
			<text class="uni-subtitle">
				<text v-if="tipMessage2">{{tipMessage2}}，</text>
				<text>用时 {{tipMessage3}}</text>
			</text>
			<br>
			<text class="uni-subtitle">{{tipMessage}}</text>
			<br v-if="tipMessage">
			<view :prop="echartOption" :echartsValue1="echartsValue1" :echartsValue2="echartsValue2"
				:change:prop="echarts.updateEcharts" :change:echartsValue1="echarts.setOption1"
				:change:echartsValue2="echarts.setOption2" id="echarts" class="echarts"></view>

		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				// value1: '50', // 每组次数
				// value2: '8', // 每次间隔
				// value3: '4', // 组数
				// value4: '180', // 每组间隔
				// value5: 0, // 是否区分左右
				currentSide: 1, // 1左 2右
				value6: 1, // 播放方式
				// value7: '3', // 左右切换时间

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
								errorMessage: '请输入次数',
							},
							{
								format: 'number',
								errorMessage: '请输入数字',
							}
						]
					},
					value2: {
						rules: [{
								required: true,
								errorMessage: '请输入次数',
							},
							{
								format: 'number',
								errorMessage: '请输入数字',
							}
						]
					},
					value3: {
						rules: [{
								required: true,
								errorMessage: '请输入次数',
							},
							{
								format: 'number',
								errorMessage: '请输入数字',
							}
						]
					},
					value4: {
						rules: [{
								required: true,
								errorMessage: '请输入次数',
							},
							{
								format: 'number',
								errorMessage: '请输入数字',
							}
						]
					},
				},

				// tts
				receiver: null,
				play: null,

				timer: null, // 组内个数计时 Interval
				timer2: null, // 组歇 timeout
				interval3: null, // 已消耗时间计时
				startTime: null, // 开始锻炼时间戳
				audio: null,
				number: 0, // 组内已完成锻炼个数
				groupNum: 0, // 已完成锻炼组数
				tipMessage: '', // 提示信息
				tipMessage2: '', // 提示信息
				tipMessage3: '00:00:00', // 已锻炼时长
				useSeconds: 0, // 锻炼已用秒数
				onStop: false,
				onStop: false, // 当前是否正在暂停标记
				onStart: false, // 当前是否正在进行锻炼标记
				audioObjAndroid: null, // android端audio对象
				audioObjH5: null, // H5端audio对象
				sideList: [{
					text: '是',
					value: 1
				}, {
					text: '否',
					value: 0
				}],
				voiceType: [{
					text: 'mp3',
					value: 0
				}, {
					text: '讯飞tts',
					value: 1
				}],
				// echarts
				echartsInterval1: null,
				echartsInterval2: null,
				echartsValue1: 0,
				echartsValue2: 0,
				gaugeData1: {},
				gaugeData2: {},
				echartOption: {},
				
				audioContext: null,
				
			}
		},
		onLoad() {
			// this.initVoicePlayer();
		},
		onReady() {
			this.getConfig();
			this.initEcartsOption();
		},
		onHide() {
			this.setKeepScreenOn(false);
		},
		onUnload() {
			this.setKeepScreenOn(false);
		},
		methods: {
			input(e) {
				// console.log('输入内容：', e);
			},
			/**
			 * 设置echarts参数默认值
			 */
			initEcartsOption() {
				this.gaugeData1 = {
					value: this.echartsValue1,
					name: '组',
					title: {
						show: false,
						offsetCenter: ['0%', '-30%']
					},
					detail: {
						valueAnimation: true,
						offsetCenter: ['0%', '-20%']
					}
				};
				this.gaugeData2 = {
					value: this.echartsValue2,
					name: '次',
					title: {
						show: false,
						offsetCenter: ['0%', '0%']
					},
					detail: {
						valueAnimation: true,
						offsetCenter: ['0%', '10%']
					}
				};
				this.echartOption = {
					series: [{
							type: 'gauge',
							name: '组',
							startAngle: 90,
							endAngle: -270,
							min: 0,
							max: Number(this.formData.value1),
							radius: '90%',
							animationEasing: 'linear',
							animationEasingUpdate: 'linear',
							animationDuration: 20, // 每组次数 / 100
							animationDurationUpdate: 20, // 每组次数 / 100
							animation: true,
							pointer: {
								show: false
							},
							progress: {
								show: true,
								overlap: false,
								roundCap: true,
								clip: false,
								itemStyle: {
									// borderWidth: 1,
									// borderColor: '#464646'
								}
							},
							axisLine: {
								lineStyle: {
									width: 20
								}
							},
							splitLine: {
								show: false,
								distance: 10,
								length: 10
							},
							axisTick: {
								show: false
							},
							axisLabel: {
								show: false,
								distance: 10
							},
							data: [this.gaugeData1],
							title: {
								fontSize: 14
							},
							detail: {
								show: false,
								width: 50,
								height: 14,
								fontSize: 14,
								color: 'inherit',
								formatter: '{value}'
							}
						},
						{
							type: 'gauge',
							name: '次',
							startAngle: 90,
							endAngle: -270,
							min: 0,
							max: Number(this.formData.value2),
							// #ifdef APP-PLUS
							radius: '55%',
							// #endif
							// #ifdef H5
							radius: '60%',
							// #endif							animationEasing: 'linear',
							animationEasingUpdate: 'linear',
							animationDuration: 20,
							animationDurationUpdate: 20,
							animation: true,
							pointer: {
								show: false
							},
							progress: {
								show: true,
								overlap: false,
								roundCap: true,
								clip: false,
								itemStyle: {
									color: '#5aabff'
									// borderWidth: 1,
									// borderColor: '#464646'
								}
							},
							axisLine: {
								lineStyle: {
									width: 20
								}
							},
							splitLine: {
								show: false,
								distance: 10,
								length: 10
							},
							axisTick: {
								show: false
							},
							axisLabel: {
								show: false,
								distance: 10
							},
							data: [this.gaugeData2],
							title: {
								fontSize: 14
							},
							detail: {
								show: false,
								width: 50,
								height: 14,
								fontSize: 14,
								color: 'inherit',
								formatter: '{value}'
							}
						}
					]
				};
			},
			/**
			 * 对表单进行校验，校验通过调用回调函数
			 * @param {Object} handler
			 */
			validateForm(handler) {
				this.$refs.form.validate().then(res => {
					// console.log('表单数据信息：', res);
					handler();
				}).catch(err => {
					console.log('表单数据信息：', err);
					uni.showToast({
						title: '配置校验失败，' + err[0].errorMessage,
						icon: 'none',
					});
				})
			},
			// 设置屏幕常亮
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
			 */
			countTimeSpend() {
				// 计算预计耗时
				let totalTime = ((Number(this.formData.value1) * (Number(this.formData.value2) + 4.5)) + (Number(this
					.formData.value4)) * Number(this.formData.value3));
				if (this.formData.value5 === 1) {
					totalTime = totalTime * 2;
					totalTime += (Number(this.formData.value7) + 3) * Number(this.formData.value3);
				}
				totalTime = (totalTime).toFixed(0); // 叠加音乐播放耗时
				this.tipMessage2 = '预计耗时 ' + totalTime + '秒';
			},
			/**
			 * 开始按钮点击
			 */
			startBtn() {
				this.validateForm(() => {
					this.stopCounter(true);
					this.setKeepScreenOn(true);
					this.setEchartOption();
					this.onStart = true;
					let musicList = [];
					if (this.formData.value5 === 1) {
						musicList.push('czcks');
					} else {
						musicList.push('kaishi');
					}
					this.playAudioList(musicList, () => {
						this.start();
					});
					this.countTimeSpend();
					this.startTimer3(true);
				});
			},
			/**
			 * 开始一组训练 次数计时
			 */
			start() {
				this.tipMessage = '运行中';
				if (this.timer) {
					clearInterval(timer);
					this.timer = null;
				}
				this.number = 0;
				this.groupNum++;

				let musicList = ['di'];
				let numList = this.splitNumber(this.groupNum);
				musicList = musicList.concat(numList);
				musicList.push('zu');
				if (this.formData.value5 === 1) {
					if (this.currentSide === 1) {
						musicList.push('zuoce');
					} else if (this.currentSide === 2) {
						musicList.push('youce');
					}
				}
				this.playAudioList(musicList, () => {
					this.startCounterIntenval(true);
				});
			},
			/** 用时统计
			 * @param {Object} startTag - 是否重新开始
			 */
			startTimer3(startTag) {
				if (startTag) {
					this.startTime = new Date().getTime();
					this.useSeconds = 0;
				}
				if (this.interval3) {
					clearInterval(this.interval3);
					this.interval3 = null;
				}
				this.interval3 = setInterval(() => {
					// const now = new Date().getTime();
					// let time = now - this.startTime;
					this.useSeconds++;
					let hour = Math.floor(this.useSeconds % (86400) / 3600);
					let minute = Math.floor(this.useSeconds % (3600) / 60);
					let second = Math.floor(this.useSeconds % (60) / 1);
					let str1 = hour > 9 ? hour + '' : hour > 0 ? '0' + hour : '00';
					let str2 = minute > 9 ? minute + '' : minute > 0 ? '0' + minute : '00';
					let str3 = second > 9 ? second + '' : second > 0 ? '0' + second : '00';
					this.tipMessage3 = str1 + ':' + str2 + ':' + str3;
				}, 1000);
			},
			/** 组内个数计时及播放
			 * @param {Object} tag - true 开始，false 继续
			 */
			startCounterIntenval(tag) {
				if (!this.onStart) {
					return;
				}
				this.startEchartsTimer1(tag);
				// this.startEchartsTimer2();
				this.startCounterIntenvalItem();
				this.timer = setInterval(() => {
					this.startCounterIntenvalItem();
				}, Number(this.formData.value2) * 1000);
			},
			startCounterIntenvalItem() {
				this.number++;
				if (this.number <= Number(this.formData.value1)) {
					this.startEchartsTimer2();
					this.playMusic();
				}
				// 完成一组，组歇时间
				if (this.number > Number(this.formData.value1)) {
					if (this.timer) {
						clearInterval(this.timer);
						this.timer = null;
					}
					this.number = 0;
					this.stopEchartsTimer1(true);
					this.stopEchartsTimer2(true);
					// 区分左右时，左侧数完再数右侧
					if (this.formData.value5 === 1 && this.currentSide === 1) {
						this.currentSide = 2;
						this.groupNum--;
						let musicList = ['zuoce', 'wancheng', 'qhdyc'];
						this.playAudioList(musicList, () => {
							// 开始组歇
							this.timer2 = setTimeout(() => {
								this.start();
							}, Number(this.formData.value7) * 1000);
						});
						return;
					} else if (this.formData.value5 === 1 && this.currentSide === 2) {
						this.currentSide = 1;
						let musicList = ['youce', 'wancheng'];
						this.playAudioList(musicList, () => {
							setTimeout(() => {
								this.groupWait();
							}, 800);
						});
						return;
					} else if (this.formData.value5 === 0) {
						// 不区分左右
						setTimeout(() => {
							this.groupWait();
						}, 800);
					}

				}
			},
			/**
			 * 每组完成，进行组歇播报或完成所有组合播报
			 */
			groupWait() {
				if (this.groupNum < Number(this.formData.value3)) {
					// 组歇播报  [您已完成,第,n,组，组歇,m,秒]
					let musicList = ['nywc', 'di'];
					let numList = this.splitNumber(this.groupNum);
					musicList = musicList.concat(numList).concat(['zu', 'zuxie']);
					let numList2 = this.splitNumber(this.formData.value4);
					musicList = musicList.concat(numList2);
					musicList.push('miao');
					setTimeout(() => {
						this.playAudioList(musicList);
					}, 5);
					// 开始组歇
					this.timer2 = setTimeout(() => {
						this.start();
					}, Number(this.formData.value4) * 1000);
					// this.playAudioList(musicList, () => {
					// 	this.timer2 = setTimeout(() => {
					// 		this.start();
					// 	}, Number(this.formData.value4) * 1000);
					// });
				} else {
					this.tipMessage = '完成所有组合';
					// ['完成所有组合', '您已完成', '共', n,组,左右,各,m,次 ]
					let musicList = ['wcsyzh', 'nywc', 'gong'];
					let numList2 = this.splitNumber(this.groupNum);
					musicList = musicList.concat(numList2);
					musicList.push('zu');
					if (this.formData.value5 === 1) {
						musicList = musicList.concat(['zuoyou', 'ge']);
					}
					let total = Number(this.formData.value1) * Number(this.groupNum);
					let numList = this.splitNumber(total);
					musicList = musicList.concat(numList);
					musicList.push('ci');
					setTimeout(() => {
						this.playAudioList(musicList);
					}, 0);
					// 取消屏幕常亮
					this.setKeepScreenOn(false);
					if (this.interval3) {
						clearInterval(this.interval3);
						this.interval3 = null;
					}
				}
			},
			/**
			 * 暂停按钮点击，暂停训练
			 */
			pause() {
				this.tipMessage = '暂停, 组歇中暂停再继续可跳过组歇';
				let musicList = ['zanting'];
				setTimeout(() => {
					this.playAudioList(musicList);
				}, 0);
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
				if (this.interval3) {
					clearInterval(this.interval3);
					this.interval3 = null;
				}
				this.onStop = true;
			},
			/**
			 * 继续按钮点击 继续训练
			 */
			resume() {
				this.validateForm(() => {
					this.tipMessage = '继续';
					let musicList = ['jixu'];
					this.setEchartOption();
					this.playAudioList(musicList, () => {
						this.startTimer3(false);
						// 组中暂停
						if (this.number !== 0) {
							this.startCounterIntenval(false);
						}
						// 组歇暂停
						if (this.number === 0) {
							this.start();
						}
					});
					this.onStop = false;
				});
			},
			/**
			 * 停止按钮点击，停止训练
			 */
			stopClick() {
				this.tipMessage = '停止';
				let musicList = ['tingzhi'];
				setTimeout(() => {
					this.playAudioList(musicList);
				}, 0);
				this.stopCounter(false);
				this.clearAudioObj();
			},
			stopCounter(resetNumTag) {
				if (resetNumTag) {
					this.number = 0;
					this.groupNum = 0;
					this.currentSide = 1;
				}
				this.startTime = null;
				this.onStart = false;
				if (this.timer) {
					clearInterval(this.timer);
					this.timer = null;
				}
				if (this.timer2) {
					clearTimeout(this.timer2);
					this.timer2 = null;
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
				let musicList = ['qingchu'];
				setTimeout(() => {
					this.playAudioList(musicList);
				}, 0)
				this.stopCounter(true);
				this.clearAudioObj();
			},
			clearAudioObj() {
				// #ifdef APP-PLUS
				if (plus.os.name === 'Android') {
					if (this.audioObjAndroid) {
						this.audioObjAndroid.pause();
						this.audioObjAndroid.stop();
						this.audioObjAndroid.destroy();
						this.audioObjAndroid = null;
					}
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
					let musicList = ['num_' + Number(num) + ''];
					setTimeout(() => {
						this.playAudioList(musicList);
					}, 5);
				} else if (num < 1000) {
					// 大于一百的整十数，组合两个音频，如110 --> 一百 一十
					let musicList = [];
					if (this.number % 10 !== 0) {
						// 取百位
						musicList.push('num_' + parseInt(num / 100) + '00');
						// 取十位
						let str = 'yishi';
						if (parseInt((num % 100) / 10) !== 1) {
							str = 'num_' + parseInt((num % 100) / 10) + '0';
						}
						musicList.push(str);
						setTimeout(() => {
							this.playAudioList(musicList);
						}, 5);
					}
				} else {
					this.tipMessage = '超过一千';
					let musicList = ['cgyqzbzc'];
					setTimeout(() => {
						this.playAudioList(musicList);
					}, 5);
					this.stop();
				}
			},

			/**
			 * 分解数字 123 --> 一百 二十 三，支持1000以下
			 * @param {Object} num
			 */
			splitNumber(num) {
				let numList = [];
				if (num > 1000) {
					numList.push('cgyqzbzc');
					return numList;
				} else {
					// 千位
					let num4 = parseInt(num / 1000);
					if (num4 > 0) {
						numList.push('num_' + num4 + '000');
					}
					// 百位
					let num3 = parseInt((num % 1000) / 100);
					if (num3 > 0) {
						numList.push('num_' + num3 + '00');
					}
					// 十位
					let num2 = parseInt((num % 100) / 10);
					if (num2 > 0) {
						numList.push('num_' + num2 + '0');
					}
					// 个位
					let num1 = parseInt(num % 10)
					if (num1 > 0) {
						numList.push('num_' + num1 + '');
					}
					return numList;
				}
			},

			/**
			 * 全部完成，共完成x次 (测试方法)
			 */
			playMusic3() {
				let numList = this.splitNumber(305);
				// console.log(numList);
				// let musicList = ['完成所有组合', '您已完成', '共', '30', this.formData.value4, '次'];
				let musicList = ['nywc', 'di', '10', 'zu', 'zuxie', '10', '6', 'miao'];
				setTimeout(() => {
					this.playAudioList(musicList);
				}, 0);
			},
			/** 
			 * 依次播放传入的音频列表
			 * @param {Object} musicList 需要播放得音频列表
			 * @param {Object} handler 播放完成后进行回调
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
			 * H5页面 用原生audio标签进行播放
			 * @param {Object} musicList  需要播放得音频列表
			 * @param {Object} handler 播放完成后进行回调
			 */
			playAudioListH5(musicList, handler) {
				try {
					if (!this.audioContext) {
						this.audioContext = new AudioContext();
					}
					if (!this.audioObjH5) {
						this.audioObjH5 = new Audio(); // 这里的路径写上mp3文件在项目中的绝对路径
					} else {
						this.audioObjH5.pause();
						this.audioObjH5 = null;
						this.audioObjH5 = new Audio();
					}
					let this_ = this;
					let musicSrc = '';
					if (musicList.length > 1) {
						let list = [];
						musicList.forEach(m => {
							list.push('/static/mp3/' + m + '.mp3');
						});
						this.concatAudioObj(list).then(src => {
							musicSrc = src;
							play(musicSrc);
						});
					} else {
						musicSrc = '/static/mp3/' + musicList[0] + '.mp3';
						play(musicSrc);
					}
					
					function play(musicSrc) {
						this_.audioObjH5.src = musicSrc;
						this_.audioObjH5.play(); // 启动音频，也就是播放
						this_.audioObjH5.addEventListener("ended", () => {
							this_.audioObjH5.pause();
							this_.audioObjH5.src = null;
							if (handler) {
								handler();
							}
						});
					}
				} catch (e) {
					//TODO handle the exception
					console.log('audioObjH5 catchError', e);
					if (this.audioObjH5) {
						this.audioObjH5.pause();
						this.audioObjH5 = null;
					}
				}
			},

			/**
			 * android 用uni-app 的audio对象进行播放
			 * @param {Object} musicList 需要播放得音频列表
			 * @param {Object} handler 播放完成后进行回调
			 */
			playAudioListAndroid(musicList, handler) {
				// console.log('musicList', musicList);
				try {
					if (!this.audioObjAndroid) {
						this.audioObjAndroid = uni.createInnerAudioContext();
					}
					let index = 0;
					let src = '/static/mp3/' + musicList[index] + '.mp3';
					this.audioObjAndroid.src = src;
					// this.audioObjAndroid.stop();
					this.audioObjAndroid.onCanplay(() => {
						this.audioObjAndroid.play(); // 启动音频，也就是播放
					});
					this.audioObjAndroid.onPlay(() => {
						// console.log('开始播放');
					});
					this.audioObjAndroid.onEnded(() => {
						index++;
						if (musicList[index]) {
							// this.audioObjAndroid.pause();
							// this.audioObjAndroid.stop();
							src = '/static/mp3/' + musicList[index] + '.mp3';
							this.audioObjAndroid.src = src;
							this.audioObjAndroid.onCanplay(() => {
								this.audioObjAndroid.play(); // 启动音频，也就是播放
							});
						} else if (this.audioObjAndroid) {
							this.audioObjAndroid.stop();
							this.audioObjAndroid.destroy();
							this.audioObjAndroid = null;
							if (handler) {
								handler();
							}
						}
					});
					this.audioObjAndroid.onError((e) => {
						console.log('audioObjAndroid onError', JSON.stringify(e), this.audioObjAndroid.src);
						if (this.audioObjAndroid) {
							this.audioObjAndroid.stop();
							this.audioObjAndroid.destroy();
							this.audioObjAndroid = null;
						}
					});
				} catch (e) {
					console.log('audioObjAndroid catchError', e);
					if (this.audioObjAndroid) {
						this.audioObjAndroid.stop();
						this.audioObjAndroid.destroy();
						this.audioObjAndroid = null;
					}
				}

			},

			/**
			 * 初始化讯飞tts语音合成工具（未完成）
			 */
			initVoicePlayer() {
				if (plus.os.name !== 'Android') {
					return;
				}
				// this.receiver = plus.android.implements('com.iflytek.cloud.SynthesizerListener', {
				// 	// onEvent: function(int eventType, int arg1, int arg2, Bundle obj) {
				// 	// 	console.log("onEvent");
				// 	// },
				// 	onSpeakBegin: function() {
				// 		console.log("开始阅读");
				// 	},
				// 	onSpeakPaused: function() {
				// 		console.log(" 暂停播放 ");
				// 	},
				// 	onSpeakResumed: function() {
				// 		console.log("继续播放");
				// 	},
				// 	onBufferProgress: function(percent, beginPos, endPos, info) {
				// 		console.log("合成进度" + percent);
				// 	},
				// 	onSpeakProgress: function(percent, beginPos, endPos) {
				// 		console.log("播放进度" + percent);
				// 	},
				// 	onCompleted: function(error) {
				// 		console.log("播放完毕");
				// 	}
				// });

				// var main = plus.android.runtimeMainActivity();

				// var SpeechUtility = plus.android.importClass('com.iflytek.cloud.SpeechUtility');

				// SpeechUtility.createUtility(main, "appid=53feacdd");

				// var SynthesizerPlayer = plus.android.importClass('com.iflytek.cloud.SpeechSynthesizer');

				// this.play = SynthesizerPlayer.createSynthesizer(main, null);


				var main = plus.android.runtimeMainActivity();
				var SpeechUtility = plus.android.importClass('com.iflytek.cloud.SpeechUtility');
				SpeechUtility.createUtility(main, 'appid=5c2c6d5f');
				var SynthesizerPlayer = plus.android.importClass('com.iflytek.cloud.SpeechSynthesizer');
				this.play = SynthesizerPlayer.createSynthesizer(main, null);
				play.startSpeaking(text, null);

			},
			/**
			 * 讯飞tts 开始合成
			 */
			startSpeaking() {
				// 开始合成
				this.play.startSpeaking('新年快乐', this.receiver);
			},
			/**
			 * 讯飞tts 取消合成
			 */
			stopSpeaking() {
				// 取消合成
				this.play.stopSpeaking();
			},
			/**
			 * 讯飞tts 暂停播放
			 */
			pauseSpeaking() {
				// 暂停播放
				this.play.pauseSpeaking();
			},
			/**
			 * 讯飞tts 继续播放
			 */
			resumeSpeaking() {
				// 继续播放
				this.play.resumeSpeaking();
			},

			/**
			 * 设置echarts通用配置
			 */
			setEchartOption() {
				this.echartOption.series[0].max = Number(this.formData.value1) * Number(this.formData.value2);
				this.echartOption.series[0].data = [this.gaugeData1];
				this.echartOption.series[0].animationDuration = Number(this.formData.value1) / 100; // 每组次数 / 100
				this.echartOption.series[0].animationDurationUpdate = Number(this.formData.value1) / 100; // 每组次数 / 100
				this.echartOption.series[1].max = Number(this.formData.value2);
				this.echartOption.series[1].data = [this.gaugeData2];
				this.echartOption = Object.assign({}, this.echartOption);
			},
			/**
			 * 组内训练个数计时器，echarts进度条渲染
			 * @param {Object} clearNum
			 */
			startEchartsTimer1(clearNum) {
				if (clearNum) {
					this.echartsValue1 = 0;
				}
				// 组
				if (this.echartsInterval1) {
					clearInterval(this.echartsInterval1);
					this.echartsInterval1 = null;
				}
				this.echartsInterval1 = setInterval(() => {
					this.echartsValue1 = this.echartsValue1 + (Number(this.formData.value1) / 100);
					if (this.echartsValue1 >= Number(this.formData.value1) * Number(this.formData.value2)) {
						this.echartsValue1 = 0;
						// console.log('===', new Date());						
					}
				}, (Number(this.formData.value1) * 1000) / 100);

			},
			/**
			 * 清除echarts 组内完成个数 进度计时器
			 * @param {Object} tag - 是否等当前圈走完了再清除
			 */
			stopEchartsTimer1(tag) {
				if (tag) {
					let interval = setInterval(() => {
						if (this.echartsValue1 === 0 && this.echartsInterval1) {
							clearInterval(this.echartsInterval1);
							this.echartsInterval1 = null;
							clearInterval(interval);
							interval = null;
						}
					}, 2);
				} else {
					if (this.echartsInterval1) {
						clearInterval(this.echartsInterval1);
						this.echartsInterval1 = null;
					}
				}

			},
			/**
			 * 每次完成进度计时器 渲染echart进度
			 */
			startEchartsTimer2() {
				if (this.echartsInterval2) {
					clearInterval(this.echartsInterval2);
					this.echartsInterval2 = null;
				}
				this.echartsValue2 = 0;
				// 次
				this.echartsInterval2 = setInterval(() => {
					this.echartsValue2 = this.echartsValue2 + 0.02;
					if (this.echartsValue2 >= Number(this.formData.value2)) {
						this.echartsValue2 = 0;
						// console.log('==', new Date().getTime());
					}
					this.echartsValue2 = Number(this.echartsValue2.toFixed(2));
				}, 20);

			},
			/**
			 * 清除echarts进度计时器 次
			 * @param {Object} tag - 是否等当前圈走完了再清除
			 */
			stopEchartsTimer2(tag) {
				if (tag) {
					let interval = setInterval(() => {
						if (this.echartsValue2 === 0 && this.echartsInterval2) {
							clearInterval(this.echartsInterval2);
							this.echartsInterval2 = null;
							clearInterval(interval);
							interval = null;
						}
					}, 2);
				} else {
					if (this.echartsInterval2) {
						clearInterval(this.echartsInterval2);
						this.echartsInterval2 = null;
					}
				}
			},
			/**
			 * 保存配置
			 */
			saveConfig() {
				let config = JSON.stringify({
					value1: this.formData.value1 || '',
					value2: this.formData.value2 || '',
					value3: this.formData.value3 || '',
					value4: this.formData.value4 || '',
					// value5: this.formData.value5 || '',
					// value7: this.formData.value7 || '',
				});
				uni.setStorage({
					key: 'rinson_toolbox_config_2',
					data: config,
					success: function() {
						// console.log('success');
						uni.showToast({
							title: '保存配置成功',
							icon: 'none',
						});
					}
				});
			},
			getConfig() {
				const value = uni.getStorageSync('rinson_toolbox_config_2');
				if (value) {
					console.log(value);
					let config = JSON.parse(value);
					if (config.value1) {
						this.formData.value1 = config.value1;
					}
					if (config.value2) {
						this.formData.value2 = config.value2;
					}
					if (config.value3) {
						this.formData.value3 = config.value3;
					}
					if (config.value4) {
						this.formData.value4 = config.value4;
					}
					// if (config.value5) {
					// 	this.formData.value5 = config.value5;
					// }
					// if (config.value7) {
					// 	this.formData.value7 = config.value7;
					// }
				}
			},
			/**
			 * 清除配置
			 */
			removeConfig() {
				uni.removeStorage({
					key: 'rinson_toolbox_config_2',
					success: function(res) {
						// console.log('success');
						uni.showToast({
							title: '清除配置成功',
							icon: 'none',
						});
					}
				});


			},

		}
	}
</script>

<script module="echarts" lang="renderjs">
	let myChart;
	let echartsValue1;
	let echartsValue2;

	export default {
		mounted() {
			if (typeof window.echarts === 'function') {
				this.initEcharts()
			} else {
				// 动态引入较大类库避免影响页面展示
				const script = document.createElement('script')
				// view 层的页面运行在 www 根目录，其相对路径相对于 www 计算
				script.src = 'static/echarts.js'
				script.onload = this.initEcharts.bind(this)
				document.head.appendChild(script)
			}
		},
		methods: {
			initEcharts() {
				setTimeout(() => {
					myChart = echarts.init(document.getElementById('echarts'))
					// 观测更新的数据在 view 层可以直接访问到
					// console.log(myChart, this.echartOption);
					// if (myChart) {
					// 	myChart.setOption(this.echartOption)
					// }
				}, 0);

			},
			updateEcharts(newValue, oldValue, ownerInstance, instance) {
				// 监听 service 层数据变更
				if (myChart) {
					// console.log(myChart);
					myChart.setOption(newValue)
				}
			},
			setOption1(newValue, oldValue, ownerInstance, instance) {
				echartsValue1 = newValue;
				// console.log('echartsValue1', echartsValue1);

				if (!myChart) {
					return;
				}
				myChart.setOption({
					series: [{
							animation: echartsValue1 !== 0,
							data: [{
								value: echartsValue1
							}]
						},
						{
							animation: echartsValue2 !== 0,
							data: [{
								value: echartsValue2
							}]
						}
					]
				});
			},
			setOption2(newValue, oldValue, ownerInstance, instance) {
				echartsValue2 = newValue;
				// console.log('echartsValue2', echartsValue2);
				if (!myChart) {
					return;
				}
				myChart.setOption({
					series: [{
							animation: echartsValue1 !== 0,
							data: [{
								value: echartsValue1
							}]
						},
						{
							animation: echartsValue2 !== 0,
							data: [{
								value: echartsValue2
							}]
						}
					]
				});
			},
			// onClick(event, ownerInstance) {
			// 	// 调用 service 层的方法
			// 	ownerInstance.callMethod('onViewClick', {
			// 		test: 'test'
			// 	})
			// }
		}
	}
</script>

<style scoped lang="scss">
	.uni-common-mt {
		padding: 20rpx 40rpx;
	}

	.form-item {
		margin-top: 20rpx;
	}

	.button-group {
		margin-top: 15px;
	}

	.rin-btn {
		display: inline-block;
		margin: 0 10px;
	}

	.echarts {
		margin-top: 10rpx;
		/* #ifdef APP-PLUS */
		width: 300rpx;
		height: 300rpx;
		/* #endif */
		/* #ifdef H5 */
		width: 250rpx;
		height: 250rpx;
		/* #endif */
	}
	
	.tipmsg-1 {
		font-size: 14px;
	}
</style>