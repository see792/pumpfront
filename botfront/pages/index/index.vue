<template>
	<view class="page-root">


		<view class="main-content">
			<view class="container">
				<view class="card">
					<view class="title">交易机器人</view>
					<view class="form-grid">
						<view class="form-group" v-for="(field, label) in fields" :key="label">
							<text class="input-label">{{ field.label }}：</text>
							<input :type="field.type" v-model="botConfig[field.model]" :placeholder="field.placeholder"
								class="z-input" />
						</view>
					</view>

					<view class="form-section">
						<text class="section-title">止盈 & 止损 设置：</text>
						<view class="stop-container">
							<view class="stop-column">
								<button class="btn" @click="addStopEarn">添加止盈</button>
								<view class="stop-group" v-for="(item, index) in botConfig.stop_earn_group"
									:key="index">

									<input type="number" v-model="item.price_percent" placeholder="涨幅百分比"
										class="z-input" />
									<input type="number" v-model="item.amount_percent" placeholder="卖出百分比"
										class="z-input" />
									<button class="btn-remove" @click="removeStopEarn(index)">删除</button>
								</view>
							</view>

							<view class="stop-column">
								<button class="btn" @click="addStopLoss">添加止损</button>
								<view class="stop-group" v-for="(item, index) in botConfig.stop_loss_group"
									:key="index">
									<input type="number" v-model="item.price_percent" placeholder="跌幅百分比"
										class="z-input" />
									<input type="number" v-model="item.amount_percent" placeholder="卖出百分比"
										class="z-input" />
									<button class="btn-remove" @click="removeStopLoss(index)">删除</button>
								</view>
							</view>
						</view>
					</view>

					<button class="btn btn-submit" @click="submitConfig">开始购买</button>
					<button class="btn btn-submit" @click="submitSell">全部卖出</button>
				</view>
			</view>
			<view class="log-container">
				<view class="title">日志</view>
				<text class="log-content">
					{{logText}}
				</text>


			</view>
		</view>

	</view>
</template>

<script>
	export default {
		data() {
			return {
				logText: "等待交易中",
				Host: "http://67.213.119.5:10911",
				botConfig: {
					pair: "",
					private_key: "",
					count: 0,
					buy_slippage: 50,
					sell_slippage: 50,
					mini_dev_sell_percent: 0,
					with_dev_sell_percent: 0,
					ray_open_percent: 0,
					stop_earn_group: [],
					stop_loss_group: []

				},
				fields: {
					pair: {
						label: "代币",
						model: "pair",
						type: "text",
						placeholder: "输入代币名称"
					},
					privateKey: {
						label: "私钥",
						model: "private_key",
						type: "text",
						placeholder: "输入私钥"
					},
					count: {
						label: "购买数量(SOL)",
						model: "count",
						type: "number",
						placeholder: "输入购买数量(SOL)"
					},
					buySlippage: {
						label: "买入滑点百分比",
						model: "buy_slippage",
						type: "number",
						placeholder: "输入买入滑点百分比"
					},
					sellSlippage: {
						label: "卖出滑点百分比",
						model: "sell_slippage",
						type: "number",
						placeholder: "输入卖出滑点百分比"
					},
					miniDevSellPercent: {
						label: "DEV卖出百分比",
						model: "mini_dev_sell_percent",
						type: "number",
						placeholder: "输入DEV卖出百分比"
					},
					withDevSellPercent: {
						label: "跟随DEV卖出百分比",
						model: "with_dev_sell_percent",
						type: "number",
						placeholder: "输入跟随DEV卖出百分比"
					},
					rayOpenPercent: {
						label: "Raydium开盘卖出百分比",
						model: "ray_open_percent",
						type: "number",
						placeholder: "输入跟随DEV卖出百分比"
					}
				}
			};
		},
		onLoad() {
			var btInfdo = uni.getStorageSync("bot")
			if (btInfdo.length > 0) {
				this.botConfig = JSON.parse(btInfdo)
			}
		},
		methods: {
			submitSell(){
				var self = this
				uni.showLoading({
					title: "卖出中",
					mask: true
				})
				uni.request({
					url: this.Host + "/api/v1/test/sell", // 请求地址
					method: 'POST', // 请求方式
					data: {
						private_key:self.botConfig.private_key
					},
					header: {
						'content-type': 'application/json' // 根据后端要求设置合适的Content-Type
					},
					success: (res) => { // 请求成功回调函数
						if (res.data.code == 1) {
							uni.showToast({
								title: "卖出成功"
							})
							return
						} else {
							uni.showToast({
								title: res.data.msg
							})
							return
						}
					},
					fail: () => {
						uni.showToast({
							title: "卖出失败"
						})
						return
					},
					complete() {
						uni.hideLoading()
					}
				})
				
			},
			addStopEarn() {
				this.botConfig.stop_earn_group.push({
					price_percent: "",
					amount_percent: ""
				});
			},
			removeStopEarn(index) {
				this.botConfig.stop_earn_group.splice(index, 1);
			},
			addStopLoss() {
				this.botConfig.stop_loss_group.push({
					price_percent: "",
					amount_percent: ""
				});
			},
			removeStopLoss(index) {
				this.botConfig.stop_loss_group.splice(index, 1);
			},
			submitConfig() {
				var bt = JSON.parse(JSON.stringify(this.botConfig))


				if (bt.pair.length < 40) {
					uni.showToast({
						title: "代币不合法"
					})
					return
				}

				if (bt.private_key.length <= 44) {
					uni.showToast({
						title: "私钥不合法"
					})
					return
				}
				if ((bt.count + "").length == 0 || bt.count <= 0) {
					uni.showToast({
						title: "请输入购买数量"
					})
					return
				}
				bt.count = parseFloat(bt.count)
				if ((bt.buy_slippage + "").length == 0 || bt.buy_slippage >= 100 || bt.buy_slippage < 0) {
					uni.showToast({
						title: "买入滑点必须0-100"
					})
					return
				}
				bt.buy_slippage = parseFloat(bt.buy_slippage) / parseFloat(100)
				if ((bt.sell_slippage + "").length == 0 || bt.sell_slippage >= 100 || bt.sell_slippage < 0) {
					uni.showToast({
						title: "卖出滑点必须0-100"
					})
					return
				}

				bt.sell_slippage = parseFloat(bt.sell_slippage) / parseFloat(100)
				if ((bt.mini_dev_sell_percent + "").length == 0 || bt.mini_dev_sell_percent > 100 || bt
					.mini_dev_sell_percent < 0) {
					uni.showToast({
						title: "DEV卖出比例必须0-100"
					})
					return
				}

				bt.mini_dev_sell_percent = parseFloat(bt.mini_dev_sell_percent) / parseFloat(100)
				if ((bt.with_dev_sell_percent + "").length == 0 || bt.with_dev_sell_percent > 100 || bt
					.with_dev_sell_percent < 0) {
					uni.showToast({
						title: "跟随DEV卖出比例必须0-100"
					})
					return
				}
				bt.with_dev_sell_percent = parseFloat(bt.with_dev_sell_percent) / parseFloat(100)
				if ((bt.ray_open_percent + "").length == 0 || bt.ray_open_percent > 100 || bt.ray_open_percent < 0) {
					uni.showToast({
						title: "Raydium开盘卖出比例必须0-100"
					})
					return
				}

				bt.ray_open_percent = parseFloat(bt.ray_open_percent) / parseFloat(100)

				for (var j in this.botConfig.stop_earn_group) {
					if ((this.botConfig.stop_earn_group[j].price_percent + "").length == 0) {
						uni.showToast({
							title: "涨幅百分比不能为空"
						})
						return
					}
					if (this.botConfig.stop_earn_group[j].price_percent <= 0) {
						uni.showToast({
							title: "涨幅百分比必须大于0"
						})
						return
					}

					if (this.botConfig.stop_earn_group[j].amount_percent <= 0) {
						uni.showToast({
							title: "止盈卖出比例必须0-100"
						})
						return
					}

					bt.stop_earn_group[j].amount_percent = parseFloat(bt.stop_earn_group[j].amount_percent) / parseFloat(
						100)

					bt.stop_earn_group[j].price_percent = parseFloat(bt.stop_earn_group[j].price_percent) / parseFloat(100)

				}

				for (var j in this.botConfig.stop_loss_group) {
					if ((this.botConfig.stop_loss_group[j].price_percent + "").length == 0) {
						uni.showToast({
							title: "跌幅百分比不能为空"
						})
						return
					}
					if (this.botConfig.stop_loss_group[j].price_percent <= 0) {
						uni.showToast({
							title: "跌幅百分比必须大于0"
						})
						return
					}

					if (this.botConfig.stop_loss_group[j].amount_percent <= 0) {
						uni.showToast({
							title: "止损卖出比例必须0-100"
						})
						return
					}


					bt.stop_loss_group[j].amount_percent = parseFloat(bt.stop_loss_group[j].amount_percent) / parseFloat(
						100)
					bt.stop_loss_group[j].price_percent = parseFloat(bt.stop_loss_group[j].price_percent) / parseFloat(100)
				}

				this.buyOp(bt)


				var bts = JSON.parse(JSON.stringify(this.botConfig))
				bts.private_key = ""
				uni.setStorageSync("bot", JSON.stringify(bts))



			},
			appendLog(nn) {
				this.logText = this.logText + "\n" + nn
			},

			buyOp(cf) {
				var self = this
				uni.showLoading({
					title: "购买中",
					mask: true
				})
				uni.request({
					url: this.Host + "/api/v1/test/buy", // 请求地址
					method: 'POST', // 请求方式
					data: cf,
					header: {
						'content-type': 'application/json' // 根据后端要求设置合适的Content-Type
					},
					success: (res) => { // 请求成功回调函数
						if (res.data.code == 1) {
							uni.showToast({
								title: "购买成功"
							})
							self.appendLog("购买" + cf.pair + "\n" + "交易:" + res.data.data + "\n" + "数量:" + cf
								.count + "SOL")
							return
						} else {
							uni.showToast({
								title: res.data.msg
							})
							return
						}
					},
					fail: () => {

						uni.showToast({
							title: "购买失败"
						})
						return
					},
					complete() {
						uni.hideLoading()
					}
				})

			},
		}
	};
</script>

<style>
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

	.page-root {
		width: 100vw;
		height: 100vh;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.log-content {
		flex: 1;
		width: auto;
		color: blueviolet;
		font-size: 10px;
		margin: 10px;
		max-height: 100vh;
		overflow-y: scroll;
		display: inline-block;
		white-space: pre-wrap;
		word-wrap: break-word;
		height: auto;
		text-align: left;

	}

	.main-content {
		display: flex;
		flex-direction: row;
		width: 1200px;
		margin: auto;
		background: #ffffff;
		padding: 30px;
		border-radius: 10px;
		box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);
		gap: 20px;
	}

	.container {
		display: flex;
		justify-content: center;

		width: 60%;
		background-color: #f4f4f4;
	}

	.log-container {

		display: flex;
		flex-direction: column;
		width: 40%;
		background: #ffffff;
		border-radius: 10px;
		box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);

	}

	.card {
		background: #ffffff;
		border-radius: 10px;
		box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);
		padding: 10px;
	}

	.title {
		font-size: 24px;
		font-weight: bold;
		margin-top: 20px;
		text-align: center;
	}

	.form-grid {
		margin-top: 40px;
		display: flex;
		flex-direction: row;
		flex-wrap: wrap;
		gap: 20px;
	}

	.form-group {
		display: flex;
		flex-direction: column;
		width: 100%;
	}

	.z-input {
		width: auto;

		margin-top: 0px;
		margin-top: 10px;
		padding: 0px;
		padding: 8px;
		border: 1px solid #ccc;
		border-radius: 5px;
		flex: 1;
	}

	.btn {
		background-color: #007bff;
		color: white;
		padding: 10px;
		border: none;
		border-radius: 5px;
		cursor: pointer;
		width: 100%;
		margin-top: 10px;
	}

	.btn-remove {
		background-color: #ff4d4f;
		width: 80px;
		height: 40px;
		margin: 0px;
		line-height: 40px;
		margin-top: 10px;
		color: white;
		font-size: 15px;
	}

	.btn-submit {
		margin-top: 20px;
		background-color: #28a745;
	}

	.stop-container {
		display: flex;
		justify-content: space-between;
		width: 100%;
	}

	.stop-column {
		width: 50%;
		background: #f9f9f9;
		padding: 5px;
		border-radius: 5px;
	}

	.sub-title {
		font-size: 16px;
		font-weight: bold;
		display: block;
		margin-bottom: 10px;
	}

	.stop-group {
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		align-items: center;

		margin-top: 10px;
		height: 40px;
		gap: 10px;

		margin-bottom: 10px;
	}

	.form-section {
		margin-top: 20px;
	}

	.section-title {
		font-size: 18px;
		font-weight: bold;
	}

	.input-label {
		font-weight: bold;
	}
</style>