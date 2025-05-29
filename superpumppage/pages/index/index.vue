<template>
	<view class="toptt">超级底部机器人</view>

	<view class="toptt2">{{getPanelInfo()}}</view>
	<view class="topBt">
		<button type="primary" class="opBt" size="mini" @click="startBuyClick()">开始购买</button>
		<button type="primary" class="opBt" size="mini" @click="stopBuyClick()">停止购买</button>
		<button type="primary" class="opBt" size="mini" @click="setCountClick()">设置数量</button>
		<button type="primary" class="opBt" size="mini" @click="allSellClick()">全部卖出</button>
	</view>
	<view class="container-parent">


		<view class="container">
			<view v-for="(item, index) in dataList" :key="index" style="margin: auto;">

				<view class="item-card" v-if="isShow(item)">
					<view class="item-header" :style="computedStyles(item)">
						<view class="head_title">
							<text class="abbreviation" @click="PumpClick( item.token )">{{ item.symbol }}</text>
							<text class="time" style="color: darkblue;">{{getFomatTimeLast(item.time)}}</text>
							<text class="time"
								style="color: blueviolet;">{{formatDateTime(new Date(item.push_time*1000),"yyyy年MM月dd日 HH:mm:ss") }}</text>
						</view>
						<image class="head_image" :src="item.image" @click="GmgnClick(item.token)"></image>
					</view>
					<view class="item-body">

						<text class="inno-text"> 购买数量:<text
								class="token-nomarl">{{ parseFloat(item.sol_count).toFixed(6) +" SOL"}}</text></text>

						<text class="inno-text"> 购买位置:<text class="token-nomarl">{{ item.buy_index }}</text></text>
						<text class="inno-text"> 购买市值:<text class="token-nomarl">${{ item.first_market }}</text></text>
						<text class="inno-text"> 当前市值:<text class="token-nomarl">${{ item.market }}</text></text>
						<text class="inno-text"> 当前盈亏:<text
								class="token-nomarl">${{parseFloat(item.income).toFixed(6) +" SOL"}}</text></text>
						<text class="inno-text"> 名字:<text class="token-nomarl">{{ item.name }}</text></text>
						<text class="inno-text"> 代币:<text class="token"
								@click="TokenClick( item.token )">{{ item.token }}</text><text class="token"
								@click="copyClick(item.token)">复制</text></text>
						<text class="inno-text"> 钱包:<text class="token"
								@click="OwnerClick( item.owner )">{{ item.owner }}</text><text class="token"
								@click="copyClick(item.owner)">复制</text></text>
						<text class="inno-text"> 提币钱包:<text class="token"
								@click="OwnerClick( item.trans.from )">{{ dealFrom(item)}}</text><text class="token"
								@click="copyClick(item.trans.from)">复制</text></text>
						<text class="inno-text"> 提币时间:<text
								class="token-nomarl">{{ getFomatTimeLast(item.trans.time) }}</text></text>

						<text class="inno-text"> 发币数:<text class="token-nomarl">{{ item.tk_count }}</text><text
								class="token" @click="lookTokenClick(item.owner)">查看</text></text>

						<text class="inno-text"> 溯源钱包:<text class="token"
								@click="OwnerClick( item.su_address)">{{ dealSuFrom(item)+"("+item.su_level+"层)"}}</text><text
								class="token" @click="copyClick(item.su_address)">复制</text></text>



						<text class="inno-text"> DEV购买:<text
								class="token-nomarl">{{ parseFloat(item.dev_sol).toFixed(6) +" SOL"}}</text></text>
						<text class="inno-text"> DEV余额:<text
								class="token-nomarl">{{ parseFloat(item.dev_balance).toFixed(6) +" SOL"}}</text></text>

						<text class="inno-text"> 推特:<text class="token"
								@click="ClickNormal( item.twitter )">{{ item.twitter }}</text>

						</text>
						<text class="inno-text"> 电报:<text class="token"
								@click="ClickNormal( item.telegram )">{{ item.telegram }}</text>
						</text>
						<text class="inno-text"> 网站:<text class="token"
								@click="ClickNormal( item.website )">{{ item.website }}</text></text>

						<view style="margin-top: 10px;"><button type="primary" class="opBt" size="mini"
								style="margin: 0px;" @click="sellClick(item.token)">卖出</button>
						</view>
					</view>
				</view>
			</view>
		</view>

	</view>







</template>

<script>
	export default {
		data() {
			return {
				// 模拟数据
				isPlay: false,
				dataList: [],
				panelInfo: {
					status: false,
					curr_balance: 0,
					total_income: 0,
					curr_count: 0
				},
				maxId: 0,
				Host: "http://bullapi.pumpfind.homes",
				walletAddresses: [],
				oldWalletAddresssed: [],
				tbTime: 0,
				isFirstLoad: true,
				showTb: 0
			};
		},
		onLoad() {
			if (false) {
				this.Host = 'http://127.0.0.1:9991'
			}

			setTimeout(() => {
				this.updateInfo()
			}, 10)



		},
		methods: {
			getPanelInfo() {
				var isOp = "关闭"
				if (this.panelInfo.status) {
					isOp = "开启"
				}
				var inc = ""
				if (this.panelInfo.total_income >= 0) {
					inc = "+" + parseFloat(this.panelInfo.total_income).toFixed(4) + ""
				} else {
					inc = parseFloat(this.panelInfo.total_income).toFixed(4) + ""
				}

				return "当前设置购买: " + parseFloat(this.panelInfo.curr_count).toFixed(4) + " SOL" + " 购买机器人: " + isOp +
					" 总余额: " + parseFloat(this.panelInfo
						.curr_balance).toFixed(4) + " SOL" + "   总盈亏: " + inc + " SOL"

			},
			onCheckChange2(e) {
				this.isPlay = e.detail.value.length > 0;
			},
			sellClick(tk) {
				uni.showLoading({
					title: "卖出中",
					mask: true
				})
				uni.request({
					url: this.Host + "/api/v1/test/sell?token=" + tk, // 请求地址
					method: 'GET', // 请求方式
					success: (res) => { // 请求成功回调函
						if (res.data.code == 1) {

							uni.showToast({
								title: "请求成功"
							})
						} else {

							uni.showToast({
								title: "请求失败"
							})
						}
						uni.hideLoading()
					},
					fail: () => {
						uni.hideLoading()
						uni.showToast({
							title: "请求失败"
						})

					}
				})
			},
			allSellClick() {
				uni.showLoading({
					title: "卖出中",
					mask: true
				})
				uni.request({
					url: this.Host + "/api/v1/test/all_token_sell", // 请求地址
					method: 'GET', // 请求方式
					success: (res) => { // 请求成功回调函
						if (res.data.code == 1) {

							uni.showToast({
								title: "请求成功"
							})
						} else {

							uni.showToast({
								title: "请求失败"
							})
						}
						uni.hideLoading()
					},
					fail: () => {
						uni.hideLoading()
						uni.showToast({
							title: "请求失败"
						})

					}
				})
			},
			startBuyClick() {
				uni.request({
					url: this.Host + "/api/v1/test/start", // 请求地址
					method: 'GET', // 请求方式
					success: (res) => { // 请求成功回调函
						if (res.data.code == 1) {

							uni.showToast({
								title: "请求成功"
							})
						} else {

							uni.showToast({
								title: "请求失败"
							})
						}
					},
					fail: () => {
						uni.showToast({
							title: "请求失败"
						})

					}
				})

			},
			stopBuyClick() {
				uni.request({
					url: this.Host + "/api/v1/test/stop", // 请求地址
					method: 'GET', // 请求方式
					success: (res) => { // 请求成功回调函
						if (res.data.code == 1) {

							uni.showToast({
								title: "请求成功"
							})
						} else {

							uni.showToast({
								title: "请求失败"
							})
						}
					},
					fail: () => {
						uni.showToast({
							title: "请求失败"
						})

					}
				})

			},

			setCountClick() {

				uni.showModal({
					title: '设置购买数量(SOL)',
					editable: true,
					content: this.panelInfo.curr_count,
					success: (res) => {
						if (res.confirm) {
							uni.request({
								url: this.Host + "/api/v1/test/set?count=" + res.content, // 请求地址
								method: 'GET', // 请求方式
								success: (res) => { // 请求成功回调函
									if (res.data.code == 1) {
										uni.showToast({
											title: "请求成功"
										})
									} else {

										uni.showToast({
											title: "请求失败"
										})
									}
								},
								fail: () => {
									uni.showToast({
										title: "请求失败"
									})

								}
							})
						}
					}
				});



			},
			isShow(it) {
				if (parseFloat(this.showTb) == 0) {
					return true
				}
				var timeNow = new Date().getTime() / 1000

				if (timeNow - it.trans.time >= parseFloat(this.showTb) * 3600) {
					return true
				} else {
					return false
				}

			},
			setTbTime(t) {
				if (t == 0) {
					this.tbTime = 0
				}
				this.showTb = t
			},
			updateInfo() {
				uni.request({
					url: this.Host + "/api/v1/test/info", // 请求地址
					method: 'GET', // 请求方式
					success: (res) => { // 请求成功回调函数
						this.isFirstLoad = false
						this.dataList = res.data.data.list
						this.panelInfo = res.data.data
						setTimeout(() => {
							this.updateInfo()
						}, 1000)

					},
					fail: () => {

						setTimeout(() => {
							this.updateInfo()
						}, 1000)
					}
				})

			},
			computedStyles2(it) {

				if (it.ten_token_counts != 0 & it.ten_token_un_counts / it.ten_token_counts > 0.5) {
					return {
						color: "red"
					}
				}
				return {
					color: "black"
				}
			},
			computedStyles(it) {
				if (it.first_market <= it.market) {
					return {
						backgroundColor: "#16ffdf"
					}
				} else {
					return {
						backgroundColor: "#ff627b"
					}
				}
				if (it.trans.time > 0) {
					var nowt = new Date().getTime() / 1000

					if (nowt - it.trans.time >= 3600 * 24 * 3) {
						return {
							backgroundColor: "#FFAAAA"
						}
					}

				}
			},
			lookTokenClick(t) {
				window.open("https://pump.fun/profile/" + t)
			},

			clearClick() {
				this.dataList = []

			},
			getFomatTimeLast: function(timestamp) {
				if (timestamp <= 0) {
					return "未知"
				}
				var mistiming = Math.round(new Date() / 1000) - timestamp;
				var postfix = mistiming > 0 ? '前' : '后'
				mistiming = Math.abs(mistiming)
				var arrr = ['年', '个月', '星期', '天', '小时', '分钟', '秒'];
				var arrn = [31536000, 2592000, 604800, 86400, 3600, 60, 1];

				for (var i = 0; i < 7; i++) {
					var inm = Math.floor(mistiming / arrn[i])
					if (inm != 0) {
						return inm + arrr[i] + postfix
					}
				}
			},
			TokenClick: function(it) {
				window.open("https://pro.xxyy.io/sol/" + it)
			},
			copyClick: function(it) {
				uni.setClipboardData({
					data: it
				})
				uni.showToast({
					title: "复制成功"
				})
			},
			dealFrom: function(it) {
				if (it.trans.from.length == "0") {
					return "未知"
				}
				if (it.memo.length > 0) {
					return it.memo
				}
				return it.trans.from

			},
			dealBlackFrom: function(it) {
				if (it.black_address.length == "0") {
					return "未知"
				}
				if (it.black_memo.length > 0) {
					return it.black_memo
				}
				return it.black_address

			},

			dealSuFrom: function(it) {
				if (it.su_address.length == 0) {
					return "未知"
				}
				if (it.su_memo.length > 0) {
					return it.su_memo
				}
				return it.su_address

			},
			OwnerClick: function(it) {
				console.log(it)
				window.open("https://solscan.io/account/" + it)
			},
			HashClick: function(it) {
				console.log(it)
				window.open("https://solscan.io/tx/" + it)
			},
			PumpClick: function(it) {
				console.log(it)
				window.open("https://pump.fun/coin/" + it)
			},
			GmgnClick: function(it) {
				console.log(it)
				window.open("https://gmgn.ai/sol/token/" + it)
			},
			ClickNormal: function(it) {
				console.log(it)
				window.open(it)
			},

			formatDateTime(date, format) {
				const o = {
					'M+': date.getMonth() + 1, // 月份
					'd+': date.getDate(), // 日
					'h+': date.getHours() % 12 === 0 ? 12 : date.getHours() % 12, // 小时
					'H+': date.getHours(), // 小时
					'm+': date.getMinutes(), // 分
					's+': date.getSeconds(), // 秒
					'q+': Math.floor((date.getMonth() + 3) / 3), // 季度
					S: date.getMilliseconds(), // 毫秒
					a: date.getHours() < 12 ? '上午' : '下午', // 上午/下午
					A: date.getHours() < 12 ? 'AM' : 'PM', // AM/PM
				};
				if (/(y+)/.test(format)) {
					format = format.replace(RegExp.$1, (date.getFullYear() + '').substr(4 - RegExp.$1.length));
				}
				for (let k in o) {
					if (new RegExp('(' + k + ')').test(format)) {
						format = format.replace(
							RegExp.$1,
							RegExp.$1.length === 1 ? o[k] : ('00' + o[k]).substr(('' + o[k]).length)
						);
					}
				}
				return format;
			},
		}
	};
</script>

<style scoped>
	page {
		background-color: #f4f7fc;
	}


	.opBt {
		width: 100px;
		margin: 10px;
	}

	.toptt {
		width: 100%;
		height: 80px;
		font-size: 40px;
		text-align: center;
		font-weight: bold;
		line-height: 80px;
		background: linear-gradient(45deg, #003366, #006699);
		color: #FFFFFF;
	}

	.toptt2 {
		width: 100%;
		height: 50px;
		font-size: 20px;
		text-align: center;
		font-weight: bold;
		line-height: 50px;
		opacity: 0.8;
		background: black;
		color: #FFFFFF;
	}

	.topBt {



		height: 80px;
		width: 100%;
		display: flex;
		flex-direction: row;
		justify-content: center;
		align-items: center;
	}

	.container-parent {
		display: flex;
		justify-content: center;
		/* 水平居中 */
		align-items: center;
		/* 垂直居中 */
		width: 100%;
		/* 占满屏幕宽度 */
	}

	.container {
		width: 100%;
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(400px, 1fr));
		/* 自适应列宽，最小宽度为200px */
		gap: 20px;
		/* 元素之间的间距 */
		padding: 20px;
	}

	/* 卡片样式 */
	.item-card {
		margin: 5px;
		width: 400px;
		height: 550px;
		/* 增大最大宽度 */
		background-color: #fff;
		border-radius: 10px;
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
		overflow: auto;
		transition: all 0.3s ease;
	}

	/* 卡片样式 */
	.item-card2 {
		width: 40%;
		/* 增大最大宽度 */
		background-color: #fff;
	}

	/* 卡片样式 */
	.item-card3 {
		flex: 1;
		/* 增大最大宽度 */
		background-color: #fff;
	}

	/* 整体滚动条 */
	::-webkit-scrollbar {
		width: 5px;
		height: 5px;
	}

	/* 滚动条轨道 */
	::-webkit-scrollbar-track {
		background: #f1f1f1;
	}

	/* 滚动条滑块 */
	::-webkit-scrollbar-thumb {
		background: #888;
		border-radius: 10px;
	}

	/* 滚动条滑块在鼠标悬停时的样式 */
	::-webkit-scrollbar-thumb:hover {
		background: #555;
	}

	.head_title {
		display: flex;
		flex-direction: column;
		text-align: left;
	}

	.head_image {
		width: 50px;
		height: 50px;
	}

	/* 卡片头部样式 */
	.item-header {
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		padding: 15px;
		background-color: #f8f9fa;
		border-bottom: 1px solid #e1e4e8;
		align-items: center;
	}

	.wallet {
		font-size: 10px;
		font-weight: bold;
		color: #333;
	}

	.abbreviation {
		font-size: 15px;
		font-weight: bold;
		text-decoration: underline;

	}

	.time {
		font-size: 15px;
		margin-top: 5px;
		color: green;
	}

	/* 卡片主体内容 */
	.item-body {
		display: flex;
		flex-direction: column;
		padding: 5px;
		background-color: #fff;
	}

	.token {
		margin-left: 5px;
		font-size: 11px;
		color: #007bff;

		text-decoration: underline;
	}

	.token-nomarl {
		margin-left: 5px;
		font-size: 11px;
		color: red;
	}

	.inno-text {
		margin-top: 5px;
		font-size: 11px;
		font-weight: bold;
	}

	/* 遮罩层 */
	.mask {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background-color: rgba(0, 0, 0, 0.5);
		display: flex;
		justify-content: center;
		align-items: center;
	}

	/* 弹窗 */
	.dialog {
		min-width: 800px;
		background-color: white;
		border-radius: 10px;
		overflow: hidden;
		box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.2);
	}

	/* 弹窗 */
	.dialog2 {
		min-width: 1400px;
		background-color: white;
		border-radius: 10px;
		overflow: hidden;
		box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.2);
	}

	/* 弹窗标题 */
	.dialog-header {
		padding: 10px;
		background-color: #f5f5f5;
		display: flex;
		justify-content: space-between;
		align-items: center;
		font-size: 18px;
	}

	/* 关闭按钮 */
	.close-btn {
		background: none;
		border: none;
		font-size: 18px;
		margin: 0px;
	}

	/* 弹窗内容 */
	.dialog-body {
		padding: 20px;
		font-size: 14px;
	}

	/* 钱包地址列表 */
	.address-list {
		margin-bottom: 20px;
		height: 400px;
	}

	.add-button {
		width: 50%;
		background-color: #007AFF;
		color: white;
		border-radius: 5px;
		padding: 10px;
		margin-top: 20px;
	}

	.address-item {
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		align-items: center;
		margin-top: 5px;
		background-color: #f9f9f9;
		border-radius: 8px;
		height: 50px;
	}

	.address-info {
		display: flex;
		flex-direction: row;
		align-items: center;
		height: 50px;
		flex: 1;
	}

	.address-name {
		font-size: 15px;
		font-weight: bold;
	}

	.address {
		color: #666;
		font-size: 15px;
		flex: 1;
		margin-left: 20px;
	}

	.blacklist-label {
		color: red;
		margin-top: 5px;
		font-size: 15px;
		font-weight: bold;
		margin-right: 20px;
	}

	.address-actions {
		display: flex;
		flex-direction: row;
	}

	.action-button {
		background-color: #007AFF;
		color: white;
		border-radius: 5px;
		font-size: 10px;
	}

	.delete-button {
		margin-left: 10px;
		background-color: #FF3B30;
	}

	.modal {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background-color: rgba(0, 0, 0, 0.5);
		display: flex;
		justify-content: center;
		align-items: center;
	}

	.modal-content {
		background-color: white;
		border-radius: 8px;
		width: 80%;
		max-width: 400px;
		padding: 20px;

	}

	.modal-header {
		text-align: center;
		margin-bottom: 20px;
	}

	.modal-title {
		font-size: 18px;
		font-weight: bold;
	}

	.modal-body .input-field {
		width: 90%;
		margin-bottom: 15px;
		padding: 10px;
		border-radius: 5px;
		border: 1px solid #ddd;
		font-size: 14px;
		margin-top: 10px;

	}

	.checkbox {
		margin-top: 10px;
	}

	.modal-footer {
		margin-top: 10px;
		display: flex;
		justify-content: space-between;
	}

	.save-button,
	.cancel-button {
		margin-bottom: 0px;
		border-radius: 5px;
		color: white;
		font-size: 16px;
	}

	.save-button {
		background-color: #28a745;
	}

	.cancel-button {
		background-color: #6c757d;
	}

	.popcontainer {
		width: 100%;
	}

	.popcontainer2 {
		display: flex;
		flex-direction: row;
		width: 100%;
	}

	.picker {
		width: 90%;
		padding: 10px;
		background-color: #f1f1f1;
		border-radius: 5px;
		font-size: 14px;
		text-align: center;
	}

	.modal-footer {
		display: flex;
		justify-content: space-between;
	}

	.tibicview {
		display: flex;
		flex-direction: row;
		align-items: center;
		justify-content: space-between;
	}

	.tibicviewinput {
		margin-left: 10px;
		margin-right: 10px;
		height: 30px;
		text-align: center;
		background-color: white;
		width: 100px;
		color: black;
		border-radius: 5px;
	}

	.holder-item {
		width: 100%;
		height: 80px;
		display: flex;
		flex-direction: row;
		justify-content: space-between;
	}

	.holder-item-left {
		display: flex;
		flex-direction: column;
		justify-content: space-around;
		text-align: left;
		margin: 5px;
	}

	.holder-item-right {
		text-align: right;
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		margin: 10px;

		flex: 1;
	}

	.holder-text {
		font-size: 15px;

	}

	.popBt {
		height: 25px;
		padding: 0px 5px 0px 5px;
		font-size: 15px;
		line-height: 25px;
		margin: 0px;
		margin-right: 5px;

	}

	.popBtView {
		display: flex;
		flex-direction: row;
	}
</style>