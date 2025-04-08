<template>
	<view class="toptt">实时代币检测</view>
	<view class="topBt">
		<button type="primary" class="opBt" @click="clearClick()">清除</button>
		<button type="primary" class="opBt" @click="addressClick()">地址管理</button>
		<view class="tibicview"> <label>提笔时间:</label> <input type="number" class="tibicviewinput" placeholder=""
				v-model="tbTime" />
			<label>小时前</label> <button type="primary" size="mini" style="margin-left: 10px;"
				@click="setTbTime(tbTime)">确定</button><button type="primary" size="mini" style="margin-left: 10px;"
				@click="setTbTime(0)">重置</button>
		</view>
	</view>
	<view style="display: flex;width: 100%;  justify-content: center;align-items: center;">
		
		
	<view class="container">
		<view v-for="(item, index) in dataList" :key="index">

			<view class="item-card" v-if="isShow(item)">
				<view class="item-header" :style="computedStyles(item)">
					<view class="head_title">
						<text class="abbreviation" @click="OwnerClick( item.owner )">{{ item.symbol }}</text>
						<text class="time" @click="OwnerClick( item.owner )">{{getFomatTimeLast(item.time)}}</text>
					</view>

					<view v-if="item.is_twitter">
						<text style="font-size: 30px; color: red;font-weight: bold;text-decoration: underline;"
							@click="twitterSearch(item)">来源推特</text>
					</view>
					<image class="head_image" :src="item.image"></image>
				</view>
				<view class="item-body">
					<text class="inno-text"> 市值:<text class="token-nomarl">${{ item.market }}</text></text>
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
					<text class="inno-text"> 推特:<text class="token"
							@click="ClickNormal( item.twitter )">{{ item.twitter }}</text>
					</text>
					<text class="inno-text"> 电报:<text class="token"
							@click="ClickNormal( item.telegram )">{{ item.telegram }}</text>
					</text>
					<text class="inno-text"> 网站:<text class="token"
							@click="ClickNormal( item.website )">{{ item.website }}</text></text>

				</view>
			</view>
		</view>

	</view>

	</view>

	<!-- 遮罩层和弹窗 -->
	<view v-if="showDialog" class="mask" @click="closeDialog">
		<view class="dialog" @click.stop>
			<view class="dialog-header">
				<text>地址管理</text>
				<button class="close-btn" @click="closeDialog">×</button>
			</view>
			<view class="dialog-body">
				<view class="popcontainer">
					<!-- 钱包地址列表 -->
					<scroll-view class="address-list" scroll-y="true">

						<view v-for="(address, index) in walletAddresses" :key="address.id" class="address-item">
							<view class="address-info">
								<view class="address-name">{{ address.name }} </view>
								<view class="address">{{ address.address }}</view>
								<view v-if="address.is_black" class="blacklist-label">黑名单</view>
							</view>
							<view class="address-actions">
								<button @click="editAddress(index)" class="action-button">编辑</button>
								<button @click="deleteAddress(index)" class="action-button delete-button">删除</button>
							</view>
						</view>
					</scroll-view>

					<button @click="openAddAddressModal" class="add-button">添加地址</button>

					<!-- 弹窗 -->
					<view v-if="showModal" class="modal">
						<view class="modal-content">
							<view class="modal-header">
								<text class="modal-title">{{ editingAddress ? '编辑地址' : '添加地址' }}</text>
							</view>
							<view class="modal-body">
								<input v-model="form.name" class="input-field" placeholder="请输入地址名称" />
								<picker mode="selector" :range="addressTypesCnssss" :value="addressTypeIndex"
									@change="onAddressTypeChange">
									<view class="picker">
										<text>{{ addressTypesCn[form.type] || '请选择地址类型' }}</text>
									</view>
								</picker>

								<input v-model="form.address" class="input-field" placeholder="请输入钱包地址" />
								<checkbox-group @change="onCheckChange">
									<checkbox :checked="form.is_black" class="checkbox">黑名单</checkbox>
								</checkbox-group>
							</view>
							<view class="modal-footer">
								<button @click="saveAddress" class="save-button">保存</button>
								<button @click="closeModal" class="cancel-button">取消</button>
							</view>
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
				addressTypes: ['exchange', 'user'],
				addressTypesCnssss: ['交易所', '普通钱包'],
				addressTypesCn: {
					"exchange": "交易所",
					"user": "普通钱包"
				},
				dataList: [],
				maxId: 0,
				Host: "http://47.90.139.134:9911",
				showDialog: false, // 控制弹窗是否显示
				walletAddresses: [],
				editingAddress: null, // 当前编辑的地址（用于新增或编辑）
				showModal: false, // 控制弹窗显示
				form: {
					name: '',
					type: '',
					is_black: true,
					address: ''
				},
				tbTime: 0,
				showTb: 0
			};
		},
		onLoad() {
			if (false) {
				this.Host = 'http://127.0.0.1:9911'
			}

			setTimeout(() => {
				this.updateInfo()
			}, 10)



		},
		methods: {
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
					url: this.Host + "/api/v1/test/info?id=" + this.maxId, // 请求地址
					method: 'GET', // 请求方式
					success: (res) => { // 请求成功回调函数
						var newdt = this.dataList.slice().reverse().concat(res.data.data)
						var newdt2 = newdt.slice().reverse()
						if (newdt2.length > 0) {
							this.maxId = newdt2[0].id
							this.dataList = newdt2
						}

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
			twitterSearch(it) {
				window.open("https://x.com/search?q=" + it.owner)
			},
			addressClick() {
				uni.request({
					url: this.Host + "/api/v1/test/address", // 请求地址
					method: 'POST', // 请求方式
					success: (res) => { // 请求成功回调函数
						this.walletAddresses = res.data.data.reverse()

					}
				})
				this.showDialog = true

			},
			computedStyles(it) {
				if (it.is_twitter) {
					return {
						backgroundColor: "yellow"
					}
				}
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
			OwnerClick: function(it) {
				console.log(it)
				window.open("https://solscan.io/account/" + it)
			},
			HashClick: function(it) {
				console.log(it)
				window.open("https://solscan.io/tx/" + it)
			},
			ClickNormal: function(it) {
				console.log(it)
				window.open(it)
			},
			closeDialog() {
				this.showDialog = false;
			},
			openAddAddressModal() {
				this.editingAddress = null;
				this.form = {
					name: '',
					type: '',
					isBlacklist: false,
					address: ''
				};
				this.showModal = true;
			},

			closeModal() {
				this.showModal = false;
			},

			saveAddress() {
				console.log(this.form)
				if (!this.form.name || !this.form.type || !this.form.address) {
					uni.showToast({
						title: '请填写所有字段',
						icon: 'none'
					});
					return;
				}

				if (this.editingAddress !== null) {
					// 编辑地址
					this.walletAddresses[this.editingAddress] = {
						...this.form,
						id: this.walletAddresses[this.editingAddress].id
					};
					console.log(this.form)
					uni.request({
						url: this.Host + "/api/v1/test/change_address", // 请求地址
						method: 'POST', // 请求方式
						header: {
							'content-type': 'application/json' // 根据后端要求设置合适的Content-Type
						},
						data: this.walletAddresses[this.editingAddress],
						success: (res) => { // 请求成功回调函数
							this.walletAddresses = res.data.data.reverse()

						}
					})
				} else {
					// 添加新地址
					const newAddress = {
						...this.form,
						id: Date.now()
					};

					uni.request({
						url: this.Host + "/api/v1/test/add_address", // 请求地址
						method: 'POST', // 请求方式
						header: {
							'content-type': 'application/json' // 根据后端要求设置合适的Content-Type
						},
						data: newAddress,
						success: (res) => { // 请求成功回调函数
							if (res.data.code != 1) {
								uni.showToast({
									title: res.data.msg
								})
							} else {

								this.walletAddresses = res.data.data.reverse()
							}

						}
					})
				}

				this.closeModal();
			},
			// 地址类型选择器改变时
			onAddressTypeChange(event) {
				console.log('选择器改变', event)
				this.addressTypeIndex = event.detail.value;
				this.form.type = this.addressTypes[this.addressTypeIndex]; // 设置选中的类型
			},

			editAddress(index) {
				this.editingAddress = index;
				const address = this.walletAddresses[index];
				this.form = {
					...address
				};
				this.showModal = true;
			},
			onCheckChange(e) {
				this.form.is_black = e.detail.value.length > 0;
			},

			deleteAddress(index) {
				uni.showModal({
					title: '确认删除',
					content: '确定删除这个地址吗？',
					success: (res) => {
						if (res.confirm) {

							uni.request({
								url: this.Host + "/api/v1/test/delete_address", // 请求地址
								method: 'POST', // 请求方式
								data: this.walletAddresses[index],
								success: (res) => { // 请求成功回调函数
									this.walletAddresses = res.data.data.reverse()

								}
							})

						}
					}
				});
			}
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
		height: 100px;
		font-size: 50px;
		text-align: center;
		font-weight: bold;
		line-height: 100px;
	}

	.topBt {



		height: 80px;
		width: 100%;
		display: flex;
		flex-direction: row;
		justify-content: center;
		align-items: center;
	}

	.container {
		background-color: #f4f7fc;
		display: flex;

		flex-direction: row;
		flex-wrap: wrap;
		align-items: center;
	}

	/* 卡片样式 */
	.item-card {
		margin: 5px;
		width: 400px;
		/* 增大最大宽度 */
		background-color: #fff;
		border-radius: 10px;
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
		overflow: hidden;
		transition: all 0.3s ease;
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
	}

	.time {
		font-size: 15px;
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
		width: 50%;
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

	/* 整体布局 */
	.container {
		padding: 20px;
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
		height: 40px;
		text-align: center;
		background-color: white;
		width: 100px;
		color: black;
		border-radius: 5px;
	}
</style>