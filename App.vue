<script>
	export default {
		onLaunch: function() {
			console.log('App Launch')
			
		},
		onShow: function() {
			console.log('App Show')
			setTimeout(() => {
				// console.log('App appId', plus.runtime.appid)
				// console.log('App launcher', plus.runtime.launcher) // 判断当前环境，是快捷方式还是默认app
				// console.log('App origin', plus.runtime.origin) // 来源
				// console.log('App arguments', plus.runtime.arguments ? JSON.parse(plus.runtime.arguments) : '') // 接受第三方app的参数，也用于接受快捷方式的启动参数
				
				const params = plus.runtime.arguments ? JSON.parse(plus.runtime.arguments).packName : ''
				if (!params) return
				// plus.runtime.quit() // 退出app
				// 通过包名打开对应app
				plus.runtime.launchApplication({
					pname: params
				},
				(e) => {
					uni.showToast({ icon: 'error', title: '启动失败：' + e.message })
				})
			}, 10)
		},
		onHide: function() {
			plus.runtime.arguments = '' // 清除参数，可保证默认app和快捷方式的启动不会冲突
			console.log('App Hide')
		}
	}
</script>

<style>
	/*每个页面公共css */
	* {
		box-sizing: border-box;
	}
</style>
