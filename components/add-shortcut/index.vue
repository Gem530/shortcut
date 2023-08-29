<template>
	<view class="mask">
		<view class="pop">
			<view class="pop-head">
				<text>创建快捷方式</text>
				<uni-icons type="closeempty" size="20" @click="close"></uni-icons>
			</view>
			<view class="old-app">
				<img class="old-app-img" v-if="currentAppInfo.appIcon" :src="currentAppInfo.appIcon">
				<view class="old-app-right">
					<view class="old-app-title">选中的应用</view>
					<view>{{currentAppInfo.appName}}</view>
				</view>
			</view>
			<view class="old-app">
				<img class="old-app-img" :src="newImg" alt="">
				<view class="old-app-right">
					<view class="old-app-title">即将创建的快捷方式</view>
					<view>{{name}}</view>
				</view>
			</view>
			<button @click="getImg">本地图片</button>
			
			<view class="canvas-box">
				<canvas class="my-canvas" canvas-id="myCanvas" id="myCanvas"></canvas>
			</view>
			<button @click="createImg">生成图标</button>
			
			<input @blur="inputBlur" class="input-name" type="text" v-model="name" placeholder="请输入快捷方式的名称">
			<button @click="toCreateShortcut">创建快捷方式</button>
		</view>
	</view>
</template>

<script>
	export default {
		props: ['currentAppInfo'],
		data() {
			return {
				name: undefined,
				newImg: undefined,
				imgUrl: undefined,
				ctx: undefined,
			}
		},
		mounted() {
			this.name = this.currentAppInfo.appName
			this.initCanvas(this.name)
		},
		methods: {
			inputBlur () {
				const that = this
				let size = 16
				if (that.name.length<4) {
					size = 20
				} else if (that.name.length<2) {
					size = 24
				}
				this.initCanvas(that.name, size)
			},
			initCanvas (name = '名称名称名称',fontSize = 16,bgColor = 'pink') {
				const that = this
				setTimeout(() => {
					let ctx = uni.createCanvasContext('myCanvas')
					
					// 填充渐变
					ctx.save()
					ctx.fillStyle=bgColor;
					ctx.fillRect(0, 0, 100, 100);
					ctx.restore()
					
					ctx.save()
					ctx.font = fontSize+'px Arial'
					ctx.fillStyle = '#fff'
					ctx.textAlign = 'center'
					ctx.translate(50, 55) // 改变中心点
					ctx.fillText(name, 0, 0)
					ctx.translate(-50, -55) // 改变中心点
					ctx.restore()
					
					ctx.draw()
					that.ctx = ctx
				}, 0)
			},
			close () {
				this.$emit('close')
			},
			getImg () {
				const that = this
				uni.chooseImage({
					success: (res) => {
						console.log(res)
						that.newImg = res.tempFilePaths[0]
						that.imgUrl = res.tempFilePaths[0]
					}
				})
			},
			createImg () {
				const that = this
				uni.canvasToTempFilePath({
					x:0,
					y:0,
					width:100,
					height:100,
					canvasId: 'myCanvas',
					success(res) {
						// console.log(res)
						const tempFilePath = res.tempFilePath
						
						uni.saveFile({
							tempFilePath,
							success(result) {
								that.imgUrl = result.savedFilePath
								// that.newImg = tempFilePath
								// that.newImg = that.ctx.toDataURL('image/png', 1)
								// console.log(result)
						
								const path = plus.io.convertLocalFileSystemURL(tempFilePath) //绝对路径
								const fileReader = new plus.io.FileReader()
								fileReader.readAsDataURL(path)
								fileReader.onloadend = (res) => { //读取文件成功完成的回调函数
									that.newImg = res.target.result
									// console.log(that.newImg) //输出base64内容
								}
							}
						})
					}
				})
			},
			hasShortcut (val) {
				const that = this
				const options = val ? val : {
					name: '快捷'+Math.random(),
					icon: that.imgUrl, // 只支持路径，base64和网络图片测试时都不行
					extra: {
						packName: 'com.iflytek.education.hnck',
						path: 'pages/index/index?name=com.iflytek.education.hnck',
						title: '快捷方式-com.iflytek.education.hnck'
					},
					toast: '',
					force: false // true 强制创建快捷方式
				}
				plus.navigator.hasShortcut(
					options,
					(e) => {
						const res = e.result
						console.log('res-', res)
						if (res == 'existing') {
							console.log('快捷方式已存在')
							// return
						}
						if (res == 'unsupported') {
							console.log('该设备不支持创建快捷方式')
							// return
						}
						// 结果类型：
						// existing 快捷方式已存在
						// none 快捷方式不存在
						// unsupported 不支持创建快捷方式
						// unknown 不确定快捷方式是否存在
						that.createShortcut(options)
					}
				)
			},
			createShortcut (options) {
				plus.navigator.createShortcut(
					options,
					(e) => {
						console.log('创建结果-success-', e)
						if (e.sure) {
							uni.showToast({ title: '创建成功' })
						}
					},
					(e) => {
						console.log('创建结果-error-', e)
					}
				)
			},
			closePop() {
				this.open = false
			},
			toCreateShortcut() {
				// console.log(val)
				const that = this
				if (!that.currentAppInfo.packageName) {
					uni.showToast({ icon: 'error', title: '未选择APP' })
					return false
				}
				const params = {
					name: that.name || that.currentAppInfo.appName,
					icon: that.imgUrl, // 只支持路径，base64和网络图片测试时都不行
					extra: {
						packName: that.currentAppInfo.packageName,
						path: 'pages/index/index?name='+that.currentAppInfo.packageName,
						title: '快捷方式-'+that.currentAppInfo.packageName
					},
					toast: '',
					force: false // true 强制创建快捷方式
				}
				that.hasShortcut(params)
			},
		}
	}
</script>

<style lang="scss" scoped>
	.mask {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		display: flex;
		justify-content: center;
		align-items: center;
		background: rgba(0, 0, 0, 0.5);
		z-index: 999;
		
		.pop {
			width: 80%;
			height: 85%;
			padding: 20rpx;
			background: #ececec;
			border-radius: 15rpx;
			overflow-y: auto;
			
			.pop-head {
				display: flex;
				justify-content: space-between;
				align-items: center;
				margin-bottom: 20rpx;
			}
			
			.old-app {
				display: flex;
				justify-content: flex-start;
				align-items: center;
				padding: 10rpx;
				margin-bottom: 20rpx;
				background: #fff;
				border-radius: 10rpx;
				
				.old-app-img {
					width: 100rpx;
					height: 100rpx;
					margin-right: 20rpx;
				}
				
				.old-app-right {}
			}
			
			.canvas-box {
				display: flex;
				justify-content: center;
				align-items: center;
				width: 100px;
				height: 100px;
				margin: 20rpx auto;
				border-radius: 20rpx;
				overflow: hidden;
				
				.my-canvas {
					width: 100px;
					height: 100px;
					// border: 1px solid #000;
				}
			}
			
			.input-name {
				height: 75rpx !important;
				padding-left: 20rpx;
				margin: 20rpx 0;
				border: 1px solid #666;
				border-radius: 16rpx;
			}
		}
	}
</style>