<template>
	<view class="content">
		<view class="radio-flex">
			<view
				:key="item.value"
				:value="item.value"
				v-for="item in tabList"
				:class="{'radio-item':true, 'active':item.value == tabIndex}"
				@click="changeAppList(item)">
				{{item.name}}
			</view>
		</view>
		<!-- 1
		<canvas canvas-id="myCanvas" id="myCanvas"></canvas>
		1
		<button @click="getImg">获取图标</button>
		<button @click="createImg">生成图标</button>
		<button @click="hasShortcut">创建快捷方式</button> -->
		<view class="app-list">
			<view class="app-item">
				<view class="app-left">已安装APP</view>
				<view class="app-right">APP快捷方式</view>
			</view>
			<view class="app-item" v-for="(item, i) in appList" :key="i">
				<view class="app-left">
					<img class="app-icon" v-if="tabIndex == 2" :src="item.appIcon" alt="">
					<text class="app-name">{{item.appName}}</text>
					<view class="app-btn" @click="toCreateShortcut(item)">点击生成快捷方式</view>
				</view>
				<view class="app-right">
					
				</view>
			</view>
		</view>
		
		<AddShortcut v-if="open" @close="closePop" :currentAppInfo="currentAppInfo"></AddShortcut>
		
		<button @click="test">测试</button>
	</view>
</template>

<script>
	// const broadcast = uni.requireNativePlugin('Ba-Broadcast')
	import AddShortcut from '@/components/add-shortcut/index.vue'
	export default {
		components: {
			AddShortcut
		},
		data() {
			return {
				open: false,
				tabIndex : 1,
				tabList: [
					{ value: 1, name: '无APP图标' },
					{ value: 2, name: '展示APP图标' },
				],
				appList: [],
				currentAppInfo: undefined,
				imgUrl: undefined
			}
		},
		onLoad() {
			// this.getAppList()
		},
		methods: {
			changeAppList (e) {
				const that = this
				that.tabIndex = e.value
				if (e.value == 2) {
					if (that.appList.length && that.appList[0].appIcon) return false
					uni.showModal({
						title: '展示APP图标',
						content: '转换APP图标需要比较长的时间，60个APP大约需要3分钟，确定展示APP图标吗？',
						cancelText: '取消',
						confirmText: '确定',
						success(res) {
							// console.log(confirm)
							if (res?.confirm) {
								that.tabIndex = 2
								that.getAppList()
								// console.log('confirm',that.tabIndex)
							} else if (res?.cancel) {
								that.tabIndex = 1
								// console.log('cancel',that.tabIndex)
							}
						}
					})
					return false
				}
			},
			async getAppList () {
				const that = this
				uni.showLoading({mask:true,title:'获取应用中'})
				plus.android.importClass('android.graphics.drawable.BitmapDrawable');
				
				// 获取已安装应用列表
				plus.android.importClass("java.util.ArrayList")
				plus.android.importClass("android.content.pm.PackageInfo")
				plus.android.importClass("android.content.pm.PackageManager")
				let applicationInfo = plus.android.importClass("android.content.pm.ApplicationInfo")
				let mainActivity = plus.android.runtimeMainActivity() // 相当于 context
				let packageManager = mainActivity.getPackageManager()
				let pinfo = plus.android.invoke(packageManager, "getInstalledPackages", 0)
				if (pinfo != null) {
					let apkList = [] // 存储应用列表
					// console.log(pinfo.size())
					for (let i = 0; i < pinfo.size(); i++) {
						let pkginfo = pinfo.get(i)
						let issysapk = ((pkginfo.plusGetAttribute("applicationInfo").plusGetAttribute("flags") & applicationInfo.FLAG_SYSTEM) != 0) ? true : false
							// console.log(applicationInfo.FLAG_SYSTEM, pkginfo.plusGetAttribute("applicationInfo").plusGetAttribute("flags"))
						if (issysapk == false) {
							const apkinfo = {
								appIco: pkginfo.plusGetAttribute("applicationInfo").loadIcon(packageManager),
								appName: pkginfo.plusGetAttribute("applicationInfo").loadLabel(packageManager).toString(),
								packageName: pkginfo.plusGetAttribute("packageName"),
								versionName: pkginfo.plusGetAttribute("versionName"),
								versionCode: pkginfo.plusGetAttribute("versionCode"),
							}
							if (that.tabIndex == 2) apkinfo.appIcon = await that.bitmapToImg(apkinfo.appIco)
							
							apkList.push(apkinfo)
						}
					}
					console.log(apkList)
					that.appList = apkList
					uni.hideLoading()
				}
			},
			bitmapToImg (apkInfoIco) {
				return new Promise((resolve, reject) => {
					try {
						// console.log(apkInfoIco)
						// 图标转换
						plus.android.importClass('android.graphics.drawable.BitmapDrawable');
						var BitmapFactory = plus.android.importClass("android.graphics.BitmapFactory");
						var Base64 = plus.android.importClass("android.util.Base64");
						var Bitmap = plus.android.importClass('android.graphics.Bitmap');
						var ByteArrayOutputStream = plus.android.importClass("java.io.ByteArrayOutputStream");
						var Canvas = plus.android.importClass('android.graphics.Canvas');
						
						// console.log('apkInfo', apkInfo)
						let bimp=null;
						try{
							bimp=apkInfoIco.getBitmap();
						}catch(e){
							bimp = Bitmap.createBitmap(apkInfoIco.getIntrinsicWidth(), apkInfoIco.getIntrinsicHeight(), Bitmap.Config.ARGB_8888);
							var canvas = new Canvas(bimp);
							apkInfoIco.setBounds(0, 0, canvas.getWidth(), canvas.getHeight());
							apkInfoIco.draw(canvas);
						}
						let baos = new ByteArrayOutputStream();
						bimp.compress(Bitmap.CompressFormat.PNG, 70, baos);
						baos.flush();
						baos.close();
						let bitmapBytes = baos.toByteArray();
						let result = "data:image/jpeg;base64,"+Base64.encodeToString(bitmapBytes, Base64.DEFAULT);
						resolve(result)
					} catch (err) {
						reject(err)
					}
				})
			},
			// hasShortcut (val) {
			// 	const that = this
			// 	const options = val ? val : {
			// 		name: '快捷'+Math.random(),
			// 		icon: that.imgUrl, // 只支持路径，base64和网络图片测试时都不行
			// 		extra: {
			// 			packName: 'com.iflytek.education.hnck',
			// 			path: 'pages/index/index?name=com.iflytek.education.hnck',
			// 			title: '快捷方式-com.iflytek.education.hnck'
			// 		},
			// 		toast: '',
			// 		force: false // true 强制创建快捷方式
			// 	}
			// 	plus.navigator.hasShortcut(
			// 		options,
			// 		(e) => {
			// 			const res = e.result
			// 			console.log('res-', res)
			// 			if (res == 'existing') {
			// 				console.log('快捷方式已存在')
			// 				// return
			// 			}
			// 			if (res == 'unsupported') {
			// 				console.log('该设备不支持创建快捷方式')
			// 				// return
			// 			}
			// 			// 结果类型：
			// 			// existing 快捷方式已存在
			// 			// none 快捷方式不存在
			// 			// unsupported 不支持创建快捷方式
			// 			// unknown 不确定快捷方式是否存在
			// 			that.createShortcut(options)
			// 		}
			// 	)
			// },
			// createShortcut (options) {
			// 	plus.navigator.createShortcut(
			// 		options,
			// 		(e) => {
			// 			console.log('创建结果-success-', e)
			// 			if (e.sure) {
			// 				uni.showToast({ title: '创建成功' })
			// 			}
			// 		},
			// 		(e) => {
			// 			console.log('创建结果-error-', e)
			// 		}
			// 	)
			// },
			closePop() {
				this.open = false
			},
			toCreateShortcut(val) {
				// console.log(val)
				const that = this
				// const params = {
				// 	name: val.appName,
				// 	icon: that.imgUrl, // 只支持路径，base64和网络图片测试时都不行
				// 	extra: {
				// 		packName: val.packageName,
				// 		path: 'pages/index/index?name='+val.packageName,
				// 		title: '快捷方式-'+val.packageName
				// 	},
				// 	toast: '',
				// 	force: false // true 强制创建快捷方式
				// }
				// that.hasShortcut(params)
				that.open = true
				that.currentAppInfo = val
			},
			toApp (val) {
				console.log(val)
				// 通过包名打开对应app
				plus.runtime.launchApplication({
					pname: val.packageName
				},
				(e) => {
					uni.showToast({ icon: 'error', title: '启动失败：' + e.message })
				})
			},
			test() {
				const that = this
				// // broadcast.sendBroadcast({
				// // 	action: 'com.iflytek.education.hnck', // 包名
				// // 	msg: 'hh', // 随意自定义参数，可多个
				// // }, res => {
				// // 	console.log(res)
				// // })
				// let Intent = plus.android.importClass('android.content.Intent')
				// let shortcut = new Intent('com.Android.launcher.action.INSTALL_SHORTCUT')
				// let intent = new Intent('com.iflytek.education.hnck') // 要给发送广播应用的包名
				// let main = plus.android.runtimeMainActivity()
				// shortcut.putExtra(Intent.EXTRA_SHORTCUT_NAME, '酷酷酷') // 名称
				// // let iconRes = Intent.ShortcutIconResource.fromContext(that, {
				// // 	"__UUID__": "Invocation29756707",
				// // 	"__TYPE__": "JSBObject",
				// // 	"className": "android.graphics.drawable.BitmapDrawable"
				// // })
				// // shortcut.putExtra(Intent.EXTRA_SHORTCUT_ICON_RESOURCE, iconRes) // 图标
				// // Invocation29756707
				// main.sendBroadcast(shortcut)
				// console.log('--', main, shortcut)
				
				
		// 		if(plus.os.name !== "Android"){ return; }  
		
		// 		var Intent = null, BitmapFactory = null;  
		// 		var main = null;  
		
		// 		// 导入要用到的类对象  
		// 		Intent = plus.android.importClass("android.content.Intent");  
		// 		BitmapFactory = plus.android.importClass("android.graphics.BitmapFactory");  
		// 		// 获取主Activity  
		// 		main = plus.android.runtimeMainActivity();        
		
		// 		// 创建快捷方式意图  
		// 		var shortcut = new Intent("com.android.launcher.action.INSTALL_SHORTCUT");  
		// 		// 设置快捷方式的名称  
		// 		shortcut.putExtra(Intent.EXTRA_SHORTCUT_NAME, 'hhh');  
		// 		// // 设置不可重复创建  
		// 		// shortcut.putExtra("duplicate", false);  
		// 		// // 设置快捷方式图标  
		// 		// var iconPath = plus.io.convertLocalFileSystemURL("static/app.png"); // 将相对路径资源转换成系统绝对路径  
		// 		// var bitmap = BitmapFactory.decodeFile(iconPath);  
		// 		// console.log(iconPath);  
		// 		// shortcut.putExtra(Intent.EXTRA_SHORTCUT_ICON,bitmap);  
		// 		// // 设置快捷方式启动执行动作  
		// 		// var action = new Intent(Intent.ACTION_MAIN);  
		// 		// action.setComponent(main.getComponentName());  
		// 		// shortcut.putExtra(Intent.EXTRA_SHORTCUT_INTENT,action);  
		// 		// 广播创建快捷方式  
		// 		main.sendBroadcast(shortcut);  
		// 		console.log( "桌面快捷方式已创建完成！" );  
		
				//获取宿主上下文
				  var main = plus.android.runtimeMainActivity();
				   //通过反射获取Android的Intent对象
				  var Intent = plus.android.importClass("android.content.Intent");
				  //通过宿主上下文创建 intent
				  var intent = new Intent(main.getIntent());
				  //设置要开启的Activity包类路径  com.HBuilder.integrate.MainActivity换掉你自己的界面
				  intent.setClassName("com.iflytek.education.hnck", "com.iflytek.education.hnck");
				  //开启新的任务栈 （跨进程）
				  intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
				  //uni向android原生界面传值
				  intent.putExtra("uni_key","来自uniapp的值");
				
				  //请求码保证了，开始的新界面和返回的是同一个操作
				  var CODE_REQUEST=1000
				  //采用startActivityForResult开启新的界面，当界面关闭时可以处理返回结果， CODE_REQUEST请求码是唯一标识
				  main.startActivityForResult(intent,CODE_REQUEST);
			},
		}
	}
</script>

<style lang="scss" scoped>
	.content {
		padding: 20rpx 30rpx;
		
		.radio-flex {
			display: flex;
			justify-content: space-between;
			align-items: center;
			margin-bottom: 50rpx;
			
			.radio-item {
				display: flex;
				justify-content: center;
				align-items: center;
				width: 48%;
				height: 60rpx;
				border: 1rpx solid #666;
				
				&.active {
					border: 1rpx solid #f00;
					color: #f00;
				}
			}
		}
		
		.app-list {
			// display: flex;
			// justify-content: space-between;
			// align-items: flex-start;
			
			.app-item {
				display: flex;
				justify-content: space-between;
				align-items: center;
				min-height: 150rpx;
				margin-bottom: 35rpx;
				.app-icon {
					width: 100rpx;
					height: 100rpx;
					margin-bottom: 10rpx;
				}
				.app-name {
					margin-bottom: 10rpx;
					font-size: 28rpx;
				}
				
				.app-left {
					display: flex;
					flex-direction: column;
					justify-content: center;
					align-items: center;
					width: 48%;
					
					.app-btn {
						padding: 5rpx 10rpx;
						border: 1rpx solid #666;
						font-size: 28rpx;
						border-radius: 6rpx;
					}
				}
				.app-right {
					display: flex;
					flex-direction: column;
					justify-content: center;
					align-items: center;
					width: 48%;
				}
			}
		}
	}
</style>
