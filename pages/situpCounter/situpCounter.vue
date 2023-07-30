<template>
	<view>
		<view class="uni-common-mt">
			<text class="uni-subtitle">本页面会播放音频,请注意音量</text>
			<view class="uni-form-item uni-column">
				<view class="form-item">
					<text class="uni-subtitle">每组次数（整数）：</text>
					<uni-easyinput class="uni-mt-5" trim="all" v-model="value1" placeholder="请输入内容"
						@input="input"></uni-easyinput>
				</view>

				<view class="form-item">
					<text class="uni-subtitle">每次时间间隔（秒，可以有小数）：</text>
					<uni-easyinput class="uni-mt-5" trim="all" v-model="value2" placeholder="请输入内容"
						@input="input"></uni-easyinput>
				</view>
				<view class="form-item">
					<text class="uni-subtitle">组数（整数）：</text>
					<uni-easyinput class="uni-mt-5" trim="all" v-model="value3" placeholder="请输入内容"
						@input="input"></uni-easyinput>
				</view>
				<view class="form-item">
					<text class="uni-subtitle">每组时间间隔（秒，可以有小数，建议8以上)：</text>
					<uni-easyinput class="uni-mt-5" trim="all" v-model="value4" placeholder="请输入内容"
						@input="input"></uni-easyinput>
				</view>
				<!-- <view class="form-item">
					<text class="uni-subtitle">是否区分左右：</text>
					<uni-data-checkbox mode="tag" v-model="value5" :localdata="sideList"></uni-data-checkbox>
				</view> -->
				<view class="form-item" v-if="value5 === 1">
					<text class="uni-subtitle">左右切换时间（单位：秒，可以有小数）：</text>
					<uni-easyinput class="uni-mt-5" trim="all" v-model="value7" placeholder="请输入内容"
						@input="input"></uni-easyinput>
				</view>
				<!-- <view class="form-item">
					<text class="uni-subtitle">播放方式：</text>
					<uni-data-checkbox mode="tag" v-model="value6" :localdata="voiceType"></uni-data-checkbox>
				</view> -->
				<!-- default -->
				<view class="button-group">
					<button type="primary" size="default" @click="startBtn">开始</button>
					<button type="primary" size="default" v-if="!onStop" @click="pause">暂停</button>
					<button type="primary" size="default" v-if="onStop" @click="resume">继续</button>
					<button type="primary" size="default" @click="stop">停止</button>
					<button type="primary" size="default" @click="countTimeSpend">计算耗时</button>
				</view>
				<!-- <view class="button-group">
					<button type="primary" size="default" @click="playMusic3">测试</button>
				</view> -->
			</view>
			<br>
			<text class="uni-subtitle">
				<text>您已完成：{{groupNum}}组，{{number}}次 </text>
				<text v-if="value5 === 1 ">{{currentSide === 1 ? '左侧' : '右侧'}}</text>
			</text>
			<br>
			<text class="uni-subtitle">{{tipMessage2}}</text>
			<br>
			<text class="uni-subtitle">{{tipMessage}}</text>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				value1: '50',
				value2: '8',
				value3: '4',
				value4: '180',
				value5: 0, // 是否区分左右
				currentSide: 1, // 1左 2右
				value6: 1,
				value7: '3',
				receiver: null,
				play: null,
				timer: null,
				timer2: null,
				audio: null,
				number: 0,
				groupNum: 0,
				tipMessage: '',
				tipMessage2: '',
				onStop: false,
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
			}
		},
		onLoad() {
			// this.initVoicePlayer();
		},
		onHide() {
			this.setKeepScreenOn(false);
		},
		onUnload() {
			this.setKeepScreenOn(false);
		},
		methods: {
			input(e) {
				console.log('输入内容：', e);
			},
			// 设置屏幕常亮
			setKeepScreenOn(tag) {
				// #ifndef H5
				uni.setKeepScreenOn({
					keepScreenOn: tag
				});
				// #endif
			},

			countTimeSpend() {
				// 计算预计耗时
				let totalTime = ((Number(this.value1) * Number(this.value2)) + Number(this.value4)) * Number(this.value3);
				if (this.value5 === 1) {
					totalTime = totalTime * 2;
					totalTime += Number(this.value7) * Number(this.value3);
				}
				totalTime = (totalTime * 1.1).toFixed(0); // 叠加音乐播放耗时
				this.tipMessage2 = '预计耗时 ' + totalTime + '秒';
			},

			startBtn() {
				this.setKeepScreenOn(true);
				let musicList = [];
				if (this.value5 === 1) {
					musicList.push('从左侧开始');
				} else {
					musicList.push('开始');
				}
				this.playAudioList(musicList, () => {
					this.start();
				});
				this.countTimeSpend();
			},
			//开始
			start() {
				this.tipMessage = '运行中';
				if (this.timer) {
					clearInterval(timer);
					this.timer = null;
				}
				this.number = 0;
				this.groupNum++;

				let musicList = ['第'];
				let numList = this.splitNumber(this.groupNum);
				musicList = musicList.concat(numList);
				musicList.push('组');
				if (this.value5 === 1) {
					if (this.currentSide === 1) {
						musicList.push('左侧');
					} else if (this.currentSide === 2) {
						musicList.push('右侧');
					}
				}
				this.playAudioList(musicList, () => {
					this.startCounterIntenval();
				});
			},
			// 组内个数计时
			startCounterIntenval() {
				this.timer = setInterval(() => {
					this.number++;
					if (this.number <= Number(this.value1)) {
						this.playMusic();
					}
					// 完成一组，组歇时间
					if (this.number > Number(this.value1)) {
						if (this.timer) {
							clearInterval(this.timer);
							this.timer = null;
						}
						this.number = 0;
						// 区分左右时，左侧数完再数右侧
						if (this.value5 === 1 && this.currentSide === 1) {
							this.currentSide = 2;
							this.groupNum--;
							let musicList = ['左侧', '完成', '切换到右侧'];
							this.playAudioList(musicList, () => {
								// 开始组歇
								this.timer2 = setTimeout(() => {
									this.start();
								}, Number(this.value7) * 1000);
							});
							return;
						} else if (this.value5 === 1 && this.currentSide === 2) {
							this.currentSide = 1;
							let musicList = ['右侧', '完成'];
							this.playAudioList(musicList, () => {
								setTimeout(() => {
									this.groupWait();
								}, 800);
							});
							return;
						} else if (this.value5 === 0) {
							// 不区分左右
							setTimeout(() => {
								this.groupWait();
							}, 800);
						}

					}
				}, Number(this.value2) * 1000);
			},
			// 组歇播报
			groupWait() {
				if (this.groupNum < Number(this.value3)) {
					// 组歇播报
					let musicList = ['您已完成', '第'];
					let numList = this.splitNumber(this.groupNum);
					musicList = musicList.concat(numList).concat(['组', '组歇']);
					let numList2 = this.splitNumber(this.value4);
					musicList = musicList.concat(numList2);
					musicList.push('秒');
					this.playAudioList(musicList);
					// 开始组歇
					this.timer2 = setTimeout(() => {
						this.start();
					}, Number(this.value4) * 1000);
					// this.playAudioList(musicList, () => {
					// 	this.timer2 = setTimeout(() => {
					// 		this.start();
					// 	}, Number(this.value4) * 1000);
					// });
				} else {
					this.tipMessage = '完成所有组合';

					let musicList = ['完成所有组合', '您已完成', '共'];
					let numList2 = this.splitNumber(this.groupNum);
					musicList = musicList.concat(numList2);
					musicList.push('组');
					if (this.value5 === 1) {
						musicList = musicList.concat(['左右', '各']);
					}
					let total = Number(this.value1) * Number(this.groupNum);
					let numList = this.splitNumber(total);
					musicList = musicList.concat(numList);
					musicList.push('次');
					this.playAudioList(musicList);
					// 取消屏幕常亮
					this.setKeepScreenOn(false);
				}
			},
			// 暂停
			pause() {
				this.tipMessage = '暂停, 组歇中暂停再继续可跳过组歇';
				let musicList = ['暂停'];
				this.playAudioList(musicList);
				clearInterval(this.timer);
				this.timer = null;
				clearTimeout(this.timer2);
				this.timer2 = null;
				this.onStop = true;
			},
			// 继续
			resume() {
				this.tipMessage = '继续';
				let musicList = ['继续'];
				this.playAudioList(musicList, () => {
					// 组中暂停
					if (this.number !== 0) {
						this.startCounterIntenval();
					}
					// 组歇暂停
					if (this.number === 0) {
						this.start();
					}
				});
				this.onStop = false;

			},
			// 停止
			stop() {
				this.tipMessage = '停止';
				let musicList = ['停止'];
				this.playAudioList(musicList);
				this.number = 0;
				this.groupNum = 0;
				clearInterval(this.timer);
				this.timer = null;
				clearTimeout(this.timer2);
				this.timer2 = null;
				this.setKeepScreenOn(false);
			},
			// 组内计数个数播放
			playMusic() {
				let num = this.number;
				if (this.number % 10 !== 0) {
					num = this.number % 10;
				}

				if (num <= 100) {
					let musicList = [Number(num) + ''];
					this.playAudioList(musicList);
				} else if (num < 1000) {
					// 大于一百的整十数，组合两个音频，如110 --> 一百 一十
					let musicList = [];
					if (this.number % 10 !== 0) {
						// 取百位
						musicList.push(parseInt(num / 100) + '00');
						// 取十位
						let str = '一十';
						if (parseInt((num % 100) / 10) !== 1) {
							str = parseInt((num % 100) / 10) + '0';
						}
						musicList.push(str);
						this.playAudioList(musicList);
					}
				} else {
					this.tipMessage = '超过一千';
					let musicList = ['超过一千暂不支持'];
					this.playAudioList(musicList);
					this.stop();
				}
			},

			// 分解数字 123 --> 一百 二十 三，支持1000以下
			splitNumber(num) {
				let numList = [];
				if (num > 1000) {
					numList.push('超过一千暂不支持');
					return numList;
				} else {
					// 千位
					let num4 = parseInt(num / 1000);
					if (num4 > 0) {
						numList.push(num4 + '000');
					}
					// 百位
					let num3 = parseInt((num % 1000) / 100);
					if (num3 > 0) {
						numList.push(num3 + '00');
					}
					// 十位
					let num2 = parseInt((num % 100) / 10);
					if (num2 > 0) {
						numList.push(num2 + '0');
					}
					// 个位
					let num1 = parseInt(num % 10)
					if (num1 > 0) {
						numList.push(num1 + '');
					}
					return numList;
				}
			},

			// 全部完成，共完成x次
			playMusic3() {
				let numList = this.splitNumber(305);
				console.log(numList);
				// let musicList = ['完成所有组合', '您已完成', '共', '30', this.value4, '次'];
				let musicList = ['您已完成', '第', '10', '组', '组歇', '10', '6', '秒'];
				this.playAudioList(musicList);
			},

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

			playAudioListH5(musicList, handler) {
				let audioObj = new Audio(); // 这里的路径写上mp3文件在项目中的绝对路径
				let index = 0;
				let src = '/static/mp3/' + musicList[index] + '.mp3';
				audioObj.src = src;
				audioObj.play(); // 启动音频，也就是播放
				audioObj.addEventListener("ended", () => {
					index++;
					if (musicList[index]) {
						let src = '/static/mp3/' + musicList[index] + '.mp3';
						audioObj.src = src;
						audioObj.play(); // 启动音频，也就是播放
					} else {
						audioObj.pause();
						audioObj = null;
						if (handler) {
							handler();
						}
					}
				});
			},

			playAudioListAndroid(musicList, handler) {
				let audioObj = uni.createInnerAudioContext();
				let index = 0;
				let src = '/static/mp3/' + musicList[index] + '.mp3';
				audioObj.src = src;
				audioObj.stop();
				audioObj.play(); // 启动音频，也就是播放
				audioObj.onEnded(() => {
					index++;
					if (musicList[index]) {
						let src = '/static/mp3/' + musicList[index] + '.mp3';
						audioObj.src = src;
						audioObj.play(); // 启动音频，也就是播放
					} else {
						audioObj.destroy();
						audioObj = null;
						if (handler) {
							handler();
						}
					}
				});
				audioObj.onError(() => {
					console.log('i am onError')
				});
			},

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

			startSpeaking() {
				// 开始合成
				this.play.startSpeaking('新年快乐', this.receiver);
			},
			stopSpeaking() {
				// 取消合成
				this.play.stopSpeaking();
			},
			pauseSpeaking() {
				// 暂停播放
				this.play.pauseSpeaking();
			},
			resumeSpeaking() {
				// 继续播放
				this.play.resumeSpeaking();
			},

		}
	}
</script>

<style scoped lang="scss">
	.uni-common-mt {
		padding: 40rpx;
	}

	.form-item {
		margin-top: 20rpx;
	}

	.button-group {
		margin-top: 15px;
		display: flex;
		justify-content: space-between;
	}
</style>