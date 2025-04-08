<template>
	<view class="toptt">实时代币检测底部聪明钱包（8个及其以上（包含机器人和>=0.8SOL,包含3个>=2SOL)  30版本）</view>
	<view class="topBt">
		<button type="primary" class="opBt" size="mini" @click="buyOp(2,1)">开始购买(不格局)</button>
		<button type="primary" class="opBt" size="mini" @click="buyOp(2,0)">停止购买(不格局)</button>
		<button type="primary" class="opBt" size="mini" @click="buyOp(1,1)">开始购买(格局)</button>
		<button type="primary" class="opBt" size="mini" @click="buyOp(1,0)">停止购买(格局)</button>


		<button type="primary" class="opBt" size="mini" @click="AllSell()">全部卖出</button>
		<button type="primary" class="opBt" size="mini" @click="tokenAnyseClick()">地址分析</button>
		<button type="primary" class="opBt" size="mini" @click="clearClick()">清除</button>
		<button type="primary" class="opBt" size="mini" @click="twitterClick()">推特管理</button>
		<button type="primary" class="opBt" size="mini" @click="addressClick()">地址管理</button>
		<view class="tibicview"> <label>提笔时间:</label> <input type="number" class="tibicviewinput" placeholder=""
				v-model="tbTime" />
			<label>小时前</label> <button type="primary" size="mini" style="margin-left: 10px;"
				@click="setTbTime(tbTime)">确定</button><button type="primary" size="mini" style="margin-left: 10px;"
				@click="setTbTime(0)">重置</button>
		</view>
		<checkbox-group @change="onCheckChange2">
			<checkbox class="checkbox" style="margin: 0px;margin-left: 10px;">播放声音</checkbox>
		</checkbox-group>
	</view>

	<view class="container-all">



		<view class="left-container">
			<text v-for="(item,index) in logArr">{{item}}</text>

		</view>
		<view class="container-parent">


			<view class="container">
				<view v-for="(item, index) in dataList" :key="index" style="margin: auto;">

					<view class="item-card" v-if="isShow(item)">
						<view class="item-header" :style="computedStyles(item)">
							<view class="head_title">
								<text class="abbreviation" @click="PumpClick( item.token )">{{ item.symbol }}</text>
								<text class="time">{{getFomatTimeLast(item.time)}}</text>
								<text class="time"
									style="color: blueviolet;">{{formatDateTime(new Date(item.push_time*1000),"yyyy年MM月dd日 HH:mm:ss") }}</text>
							</view>
							<view v-if="item.is_search">
								<text style="font-size: 30px; color: red;font-weight: bold;text-decoration: underline;"
									@click="twitterTokenSearch(item)">推特CA</text>

							</view>
							<view v-if="item.is_twitter">
								<text style="font-size: 30px; color: red;font-weight: bold;text-decoration: underline;"
									@click="twitterSearch(item)">推特钱包</text>
							</view>
							<image class="head_image" :src="item.image"></image>
						</view>
						<view class="item-body">

							<text v-if="item.is_twitter||item.is_search " class="inno-text"> 发推人:<text class="token"
									@click="ClickNormal( 'https://x.com/'+item.twitter_info.user.screen_name )">{{ getTwitterMemo(item)}}</text>
								<text class="token" @click="blackTwitterClick(item)">拉黑</text></text>

							<text v-if="item.is_twitter||item.is_search " class="inno-text"> 粉丝数:<text
									class="token-nomarl">{{ item.twitter_info.user.followers_count}}</text>
							</text>

							<text class="inno-text"> 市值:<text class="token-nomarl">${{ item.market }}</text></text>
							<text class="inno-text"> 名字:<text class="token-nomarl">{{ item.name }}</text></text>
							<text class="inno-text"> 代币:<text class="token"
									@click="TokenClick( item.token )">{{ item.token }}</text><text class="token"
									@click="copyClick(item.token)">复制</text></text>
							<text class="inno-text"> 钱包:<text class="token"
									@click="OwnerClick( item.owner )">{{ item.owner }}</text><text class="token"
									@click="copyClick(item.owner)">复制</text></text>
							<text class="inno-text"> 发币数:<text class="token-nomarl">{{ item.tk_count }}</text><text
									class="token" @click="lookTokenClick(item.owner)">查看</text></text>
							<text class="inno-text"> 提币钱包:<text class="token"
									@click="OwnerClick( item.trans.from )">{{ dealFrom(item)}}</text><text class="token"
									@click="copyClick(item.trans.from)">复制</text></text>

							<text class="inno-text"> 溯源钱包:<text class="token"
									@click="OwnerClick( item.su_address)">{{ dealSuFrom(item)+"("+item.su_level+"层)"}}</text><text
									class="token" @click="copyClick(item.su_address)">复制</text></text>
							<text class="inno-text"> 提币时间:<text
									class="token-nomarl">{{ getFomatTimeLast(item.trans.time) }}</text></text>
							<text class="inno-text"> 推特:<text class="token"
									@click="ClickNormal( item.twitter )">{{ item.twitter }}</text>
								<text class="token" @click="blackTwitterClick2(item)"
									v-if="item.twitter.length>0">拉黑</text>
							</text>
							<text class="inno-text"> 电报:<text class="token"
									@click="ClickNormal( item.telegram )">{{ item.telegram }}</text>
							</text>
							<text class="inno-text"> 网站:<text class="token"
									@click="ClickNormal( item.website )">{{ item.website }}</text></text>

							<view style="margin-top: 10px;"><button type="primary" class="opBt" size="mini"
									style="margin: 0px;" @click="analyseToken(item.token)">代币分析</button>
							</view>
						</view>
					</view>
				</view>
			</view>

		</view>



	</view>

	<!-- 遮罩层和弹窗 -->
	<view v-if="showAddressDialog" class="mask" @click="closeAddressDialog">
		<view class="dialog2" @click.stop>
			<view class="dialog-header">
				<text>地址分析</text>
				<button class="close-btn" @click="closeAddressDialog">×</button>
			</view>
			<view class="dialog-body" style="padding: 1px;">
				<view class="popcontainer2">
					<view style="width: 2px;background-color: chocolate;"></view>
					<view class="item-card3">
						<view class="holder-item">
							<view class="holder-item-left">
								<text>{{adddressInfo.tag}}</text>
								<text></text>
								<text>
									<text class="token" style="font-size: 15px;margin: 0px;"
										@click="OwnerClick(adddressInfo.address)">{{dealTag(adddressInfo)}}</text><text
										class="token" @click="copyClick(adddressInfo.address)">复制</text></text>
								<text></text>
								<view class="popBtView">
									<button type="primary" class="popBt"
										@click="memoCLick(adddressInfo,'备注')">备注</button>
									<button type="primary" class="popBt"
										@click="memoCLick(adddressInfo,'诈骗')">诈骗</button>
									<button type="primary" class="popBt"
										@click="memoCLick(adddressInfo,'聪明钱')">聪明钱</button>
									<button type="primary" class="popBt"
										@click="memoCLick(adddressInfo,'机器人')">机器人</button>
									<button type="primary" class="popBt"
										@click="memoCLick(adddressInfo,'笨蛋钱')">笨蛋钱</button>
									<button type="primary" class="popBt"
										@click="memoCLick(adddressInfo,'正常人')">正常人</button>
								</view>
								<text></text>
							</view>
							<view class="holder-item-right">

								<text class="holder-text">余额:<text
										style="color: red;">{{" "+adddressInfo.balance.toFixed(2)}}SOL</text></text>

								<text class="holder-text" style="font-size: 20px;font-weight: bold;">10天代币:{{" "}}
									<text
										:style="computedStyles2(adddressInfo)">{{adddressInfo.ten_token_un_counts}}</text><text>/{{adddressInfo.ten_token_counts}}{{detailPercent(adddressInfo)}}</text></text>
							</view>

						</view>
						<view style="width: 100%;height: 1px; background-color: red;"></view>
					</view>
				</view>
			</view>
		</view>
	</view>





	<!-- 遮罩层和弹窗 -->
	<view v-if="showAnyDialog" class="mask" @click="closeAnyDialog">
		<view class="dialog2" @click.stop>
			<view class="dialog-header">
				<text>代币分析</text>
				<button class="close-btn" @click="closeAnyDialog">×</button>
			</view>
			<view class="dialog-body" style="padding: 1px;">
				<view class="item-header">
					<view class="head_title">
						<text class="abbreviation" @click="PumpClick( tkInfo.token )">{{ tkInfo.symbol }}</text>
						<text class="time">{{getFomatTimeLast(tkInfo.time)}}</text>
					</view>

					<view v-if="tkInfo.is_twitter">
						<text style="font-size: 30px; color: red;font-weight: bold;text-decoration: underline;"
							@click="twitterSearch(tkInfo)">推特钱包</text>
					</view>
					<image class="head_image" :src="tkInfo.image"></image>
				</view>
				<view class="popcontainer2">

					<view class="item-card2">
						<view class="item-body">
							<text class="inno-text" style="font-size: 15px;">
								市值:
								<text class="token-nomarl" style="font-size: 15px;">${{ tkInfo.market }}</text>
							</text>
							<view style="margin-top: 20px;"></view>

							<text class="inno-text" style="font-size: 15px;">
								名字:
								<text class="token-nomarl" style="font-size: 15px;">{{ tkInfo.name }}</text>
							</text>
							<view style="margin-top: 20px;"></view>

							<text class="inno-text" style="font-size: 15px;">
								代币:
								<text class="token" @click="TokenClick(tkInfo.token)"
									style="font-size: 15px;">{{ tkInfo.token }}</text>
								<text class="token" @click="copyClick(tkInfo.token)" style="font-size: 15px;">复制</text>
							</text>
							<view style="margin-top: 20px;"></view>

							<text class="inno-text" style="font-size: 15px;">
								钱包:
								<text class="token" @click="OwnerClick(tkInfo.owner)"
									style="font-size: 15px;">{{ tkInfo.owner }}</text>
								<text class="token" @click="copyClick(tkInfo.owner)" style="font-size: 15px;">复制</text>
							</text>
							<view style="margin-top: 20px;"></view>

							<text class="inno-text" style="font-size: 15px;">
								提币钱包:
								<text class="token" @click="OwnerClick(tkInfo.trans.from)"
									style="font-size: 15px;">{{ dealBlackFrom(tkInfo) }}</text>
								<text class="token" @click="copyClick(tkInfo.trans.from)"
									style="font-size: 15px;">复制</text>
							</text>
							<view style="margin-top: 20px;"></view>

							<text class="inno-text" style="font-size: 15px;">
								提币时间:
								<text class="token-nomarl"
									style="font-size: 15px;">{{ getFomatTimeLast(tkInfo.trans.time) }}</text>
							</text>
							<view style="margin-top: 20px;"></view>

							<text class="inno-text" style="font-size: 15px;">
								推特:
								<text class="token" @click="ClickNormal(tkInfo.twitter)"
									style="font-size: 15px;">{{ tkInfo.twitter }} </text>
								<text class="token" @click="blackTwitterClick2(tkInfo)"
									v-if="tkInfo.twitter.length>0">拉黑</text>
							</text>
							<view style="margin-top: 20px;"></view>

							<text class="inno-text" style="font-size: 15px;">
								电报:
								<text class="token" @click="ClickNormal(tkInfo.telegram)"
									style="font-size: 15px;">{{ tkInfo.telegram }}</text>
							</text>
							<view style="margin-top: 20px;"></view>

							<text class="inno-text" style="font-size: 15px;">
								网站:
								<text class="token" @click="ClickNormal(tkInfo.website)"
									style="font-size: 15px;">{{ tkInfo.website }}</text>
							</text>
							<view style="margin-top: 20px;"></view>
							<text class="inno-text" style="font-size: 15px;">
								前十小草:
								<text class="token-nomarl" style="font-size: 15px;">{{ tkInfo.small_ten }}</text>
							</text>
							<view style="margin-top: 20px;"></view>
							<text class="inno-text" style="font-size: 15px;">
								前五小草:
								<text class="token-nomarl" style="font-size: 15px;">{{ tkInfo.small_five }}</text>
							</text>

							<view style="margin-top: 20px;"></view>

							<text class="inno-text" style="font-size: 15px;">
								黑名单交互:
								<text class="token" @click="OwnerClick(tkInfo.black_address)"
									style="font-size: 15px;">{{ dealBlackFrom(tkInfo) }}</text>



								<text class="token" @click="copyClick(tkInfo.black_address)"
									style="font-size: 15px;">复制</text>
							</text>

							<view style="margin-top: 20px;"></view>
						</view>

					</view>
					<view style="width: 2px;background-color: chocolate;"></view>
					<view class="item-card3">
						<view v-for="(holder ,index) in tkInfo.list">

							<view class="holder-item">
								<view class="holder-item-left">
									<text>{{index+1}}</text>
									<text>
										<text class="token" style="font-size: 15px;margin: 0px;"
											@click="OwnerClick(holder.address)">{{dealTag(holder)}}</text><text
											class="token" @click="copyClick(holder.address)">复制</text></text>
									<view class="popBtView">
										<button type="primary" class="popBt" @click="memoCLick(holder,'备注')">备注</button>
										<button type="primary" class="popBt" @click="memoCLick(holder,'诈骗')">诈骗</button>
										<button type="primary" class="popBt"
											@click="memoCLick(holder,'聪明钱')">聪明钱</button>
										<button type="primary" class="popBt"
											@click="memoCLick(holder,'机器人')">机器人</button>
										<button type="primary" class="popBt"
											@click="memoCLick(holder,'笨蛋钱')">笨蛋钱</button>

										<button type="primary" class="popBt"
											@click="memoCLick(holder,'正常人')">正常人</button>
									</view>
								</view>
								<view class="holder-item-right">

									<text class="holder-text">余额:<text
											style="color: red;">{{" "+holder.balance.toFixed(2)}}SOL</text></text>

									<text class="holder-text">持仓:<text
											style="color: red;">{{" "+holder.token_balance}}{{" "+tkInfo.symbol}}({{(parseFloat(holder.token_balance)/parseFloat(1000000000)*100).toFixed(2)}}%)</text></text>

									<text class="holder-text" style="font-size: 20px;font-weight: bold;">10天代币:{{" "}}
										<text
											:style="computedStyles2(holder)">{{holder.ten_token_un_counts}}</text><text>/{{holder.ten_token_counts}}{{detailPercent(holder)}}</text></text>
								</view>

							</view>
							<view style="width: 100%;height: 1px; background-color: red;"></view>
						</view>
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
				<view class="tibicview"> <label>钱包地址或者备注:</label> <input class="tibicviewinput" style="width: 400px;"
						placeholder="" v-model="vbsearch" />
					<button type="primary" size="mini" style="margin-left: 10px;"
						@click="searchAddress()">搜索</button><button type="primary" size="mini"
						style="margin-left: 10px;" @click="resetSearch()">重置</button>
				</view>
				<button class="close-btn" @click="closeDialog">×</button>
			</view>
			<view class="dialog-body">
				<view class="popcontainer">
					<!-- 钱包地址列表 -->
					<scroll-view class="address-list" scroll-y="true">

						<view v-for="(address, index) in walletAddresses.slice(0, 200)" :key="address.id"
							class="address-item">
							<view class="address-info">
								<view class="address-name">{{ address.name }} </view>
								<view class="address">{{ address.address }}</view>

								<view class="address">{{addressTypesCn[address.type] }}</view>
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






	<!-- 遮罩层和弹窗 -->
	<view v-if="showTwitterDialog" class="mask" @click="closeTwitterDialog">
		<view class="dialog" @click.stop>
			<view class="dialog-header">
				<text>推特管理</text>
				<view class="tibicview"> <label>推特:</label> <input class="tibicviewinput" style="width: 400px;"
						placeholder="推特名字或者备注" v-model="twitter_search" />
					<button type="primary" size="mini" style="margin-left: 10px;"
						@click="searchTwitter()">搜索</button><button type="primary" size="mini"
						style="margin-left: 10px;" @click="resetTwitterSearch()">重置</button>
				</view>
				<button class="close-btn" @click="closeTwitterDialog">×</button>
			</view>
			<view class="dialog-body">
				<view class="popcontainer">
					<!-- 钱包地址列表 -->
					<scroll-view class="address-list" scroll-y="true">

						<view v-for="(twitter, index) in twitterList" :key="twitter.id" class="address-item">
							<view class="address-info">
								<view class="address-name">{{ twitter.name }} </view>
								<view class="address">{{ twitter.twitter }}</view>
								<view v-if="twitter.is_black" class="blacklist-label">黑名单</view>
							</view>
							<view class="address-actions">
								<button @click="editTwitter(index)" class="action-button">编辑</button>
								<button @click="deleteTwitter(index)" class="action-button delete-button">删除</button>
							</view>
						</view>
					</scroll-view>

					<button @click="openAddTwitterModal" class="add-button">添加推特</button>

					<!-- 弹窗 -->
					<view v-if="showTwitterModal" class="modal">
						<view class="modal-content">
							<view class="modal-header">
								<text class="modal-title">{{ editingTwitter ? '编辑推特' : '添加推特' }}</text>
							</view>
							<view class="modal-body">
								<input v-model="twitter_form.name" class="input-field" placeholder="请输入备注" />

								<input v-model="twitter_form.twitter" class="input-field" placeholder="请输入推特名字" />
								<checkbox-group @change="onCheckTwChange">
									<checkbox :checked="twitter_form.is_black" class="checkbox">黑名单</checkbox>
								</checkbox-group>
							</view>
							<view class="modal-footer">
								<button @click="saveTwitter()" class="save-button">保存</button>
								<button @click="closeTwModal()" class="cancel-button">取消</button>
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
				addressTypes: ['exchange', 'user', 'pump'],
				addressTypesCnssss: ['交易所', '普通钱包', '流动分析'],
				addressTypesCn: {
					"exchange": "交易所",
					"user": "普通钱包",
					"pump": "流动分析"
				},
				isPlay: false,
				dataList: [],
				maxId: 0,
				showAnyDialog: false,
				showAddressDialog: false,
				twitter_search: "",
				showTwitterDialog: false,
				adddressInfo: {},
				tkInfo: {},
				Host: "http://api2.pumpfind.homes",
				showDialog: false, // 控制弹窗是否显示
				walletAddresses: [],
				oldWalletAddresssed: [],
				twitterList: [],
				oldTwitterList: [],
				editingAddress: null, // 当前编辑的地址（用于新增或编辑）
				editingTwitter: null, // 当前编辑的地址（用于新增或编辑）
				showModal: false, // 控制弹窗显示
				showTwitterModal: false, // 控制弹窗显示
				form: {
					name: '',
					type: '',
					is_black: true,
					address: ''
				},

				twitter_form: {
					name: '',
					is_black: true,
					twitter: ''
				},
				vbsearch: "",
				tbTime: 0,
				isFirstLoad: true,
				showTb: 0,
				logArr: ["购买日志"]
			};
		},
		onLoad() {
			if (false) {
				this.Host = 'http://127.0.0.1:6911'
			}

			setTimeout(() => {
				this.updateInfo()
			}, 10)



		},
		methods: {
			AllSell() {
				console.log("准备全部卖出")
				var self = this
				uni.showLoading({
					title: "卖出中",
					mask: true
				})
				uni.request({
					url: this.Host + "/api/v1/test/all_sell", // 请求地址
					method: 'POST', // 请求方式

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
			buyOp(bt, isp) {
				uni.request({
					url: this.Host + "/api/v1/test/op_robot?open=" + isp + "&bot=" + bt, // 请求地址
					method: 'GET', // 请求方式
					success: (res) => { // 请求成功回调函数
						uni.showToast({
							title: "操作成功"
						})
						return

					},
					fail: () => {

						uni.showToast({
							title: "操作失败"
						})
						return
					}
				})

			},

			getTwitterMemo(item) {

				if (item.twitter_memo.length > 0) {
					return item.twitter_memo
				}
				return item.twitter_info.user.screen_name

			},

			blackTwitterClick2(item) {
				var tt = ""
				var lastIndex = item.twitter.lastIndexOf("/")
				if (lastIndex > 0) {

					tt = item.twitter.substring(lastIndex + 1)

				} else {
					uni.showToast({
						title: "无效推特"
					})
					return
				}


				uni.request({
					url: this.Host + "/api/v1/test/add_twitter", // 请求地址
					method: 'POST', // 请求方式
					header: {
						'content-type': 'application/json' // 根据后端要求设置合适的Content-Type
					},
					data: {
						name: tt,
						is_black: true,
						twitter: tt
					},
					success: (res) => { // 请求成功回调函数
						if (res.data.code != 1) {
							uni.showToast({
								title: res.data.msg
							})
						} else {
							uni.showToast({
								title: "拉黑成功",
							})
							this.twitterList = res.data.data.reverse()

							this.oldTwitterList = this.twitterList.concat([])

						}

					}
				})
			},

			blackTwitterClick(item) {


				var tt = item.twitter_info.user.screen_name

				uni.request({
					url: this.Host + "/api/v1/test/add_twitter", // 请求地址
					method: 'POST', // 请求方式
					header: {
						'content-type': 'application/json' // 根据后端要求设置合适的Content-Type
					},
					data: {
						name: item.twitter_info.user.screen_name,
						is_black: true,
						twitter: item.twitter_info.user.screen_name
					},
					success: (res) => { // 请求成功回调函数
						if (res.data.code != 1) {
							uni.showToast({
								title: res.data.msg
							})
						} else {
							uni.showToast({
								title: "拉黑成功",
							})
							this.twitterList = res.data.data.reverse()

							this.oldTwitterList = this.twitterList.concat([])

						}

					}
				})
			},

			searchTwitter() {
				if (this.twitter_search.trim().length == 0) {
					uni.showToast({
						title: "请输入内容"
					})
					return
				}
				var addr = []
				for (var j in this.oldTwitterList) {
					if (this.oldTwitterList[j].twitter.includes(this.twitter_search) || this.oldTwitterList[j].name
						.includes(this.twitter_search)) {
						addr.push(this.oldTwitterList[j])
					}
				}
				this.twitterList = addr
			},
			resetTwitterSearch() {
				this.twitter_search = ""
				this.twitterList = this.oldTwitterList.concat([])

			},

			searchAddress() {
				if (this.vbsearch.trim().length == 0) {
					uni.showToast({
						title: "请输入内容"
					})
					return
				}
				var addr = []
				for (var j in this.oldWalletAddresssed) {
					if (this.oldWalletAddresssed[j].address.includes(this.vbsearch) || this.oldWalletAddresssed[j].name
						.includes(this.vbsearch)) {

						addr.push(this.oldWalletAddresssed[j])
					}
				}
				this.walletAddresses = addr
			},
			resetSearch() {
				this.vbsearch = ""
				this.walletAddresses = this.oldWalletAddresssed.concat([])

			},
			dealTag(h1) {
				if (h1.is_robot) {
					return h1.address + "(" + h1.tag + ")"
				}
				if (h1.tag.length > 0) {
					return h1.tag
				}
				return h1.address
			},
			memoCLick(h1, mstr) {
				if (mstr == "备注") {
					uni.showModal({
						title: '修改备注',
						content: '',
						placeholderText: "请输入备注名称",
						confirmText: "修改",
						editable: true,
						success: (rescc) => {
							if (rescc.confirm) {

								uni.request({
									url: this.Host + "/api/v1/test/add_address", // 请求地址
									method: 'POST', // 请求方式
									header: {
										'content-type': 'application/json' // 根据后端要求设置合适的Content-Type
									},
									data: {
										name: rescc.content,
										type: 'pump',
										is_black: false,
										address: h1.address

									},
									success: (res) => { // 请求成功回调函数
										if (res.data.code != 1) {
											uni.showToast({
												title: res.data.msg
											})
										} else {
											h1.tag = rescc.content
										}

									}
								})
							} else if (rescc.cancel) {
								console.log('用户点击取消');
							}
						}
					});
				} else {

					if (mstr == "诈骗") {
						uni.request({
							url: this.Host + "/api/v1/test/add_address", // 请求地址
							method: 'POST', // 请求方式
							header: {
								'content-type': 'application/json' // 根据后端要求设置合适的Content-Type
							},
							data: {
								name: h1.address.substring(0, 4) + mstr,
								type: 'user',
								is_black: true,
								address: h1.address

							},
							success: (res) => { // 请求成功回调函数
								if (res.data.code != 1) {
									uni.showToast({
										title: res.data.msg
									})
								} else {
									h1.tag = h1.address.substring(0, 4) + mstr
								}

							}
						})
					} else {
						uni.request({
							url: this.Host + "/api/v1/test/add_address", // 请求地址
							method: 'POST', // 请求方式
							header: {
								'content-type': 'application/json' // 根据后端要求设置合适的Content-Type
							},
							data: {
								name: (h1.address + "").substring(0, 4) + mstr,
								type: 'pump',
								is_black: false,
								address: h1.address

							},
							success: (res) => { // 请求成功回调函数
								if (res.data.code != 1) {
									uni.showToast({
										title: res.data.msg
									})
								} else {
									h1.tag = h1.address.substring(0, 4) + mstr
								}

							}
						})
					}





				}



			},

			tokenAnyseClick() {
				uni.showModal({
					title: '分析地址',
					content: '',
					placeholderText: "请输入地址",
					confirmText: "开始",
					editable: true,
					success: (res) => {
						if (res.confirm) {
							console.log('用户点击确定');
							console.log(res.content)
							this.analyseToken(res.content)

						} else if (res.cancel) {
							console.log('用户点击取消');
						}
					}
				});
			},
			analyseToken(tk) {

				uni.showLoading({
					title: "分析中",
					mask: true
				})
				uni.request({
					url: this.Host + "/api/v1/test/detail?address=" + tk, // 请求地址
					method: 'GET', // 请求方式
					timeout: 120000,
					success: (res) => { // 请求成功回调函数

						uni.hideLoading()
						if (res.data.code == 1) {
							if ((res.data.data.token + "").length > 10) {
								this.tkInfo = res.data.data
								this.showAnyDialog = true
							} else {
								this.adddressInfo = res.data.data
								this.showAddressDialog = true
							}

						} else {
							uni.showToast({
								title: res.data.msg
							})
						}
					},
					fail: () => {
						uni.hideLoading()
					}
				})

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
					url: this.Host + "/api/v1/test/robot_log", // 请求地址
					method: 'GET', // 请求方式
					success: (res) => { // 请求成功回调函数
						if (res.data.code == 1) {
							var st = "开启"
							if (!res.data.data.open) {
								st = "关闭"
							}
							var st2 = "开启"
							if (!res.data.data.open2) {
								st2 = "关闭"
							}
							var logsrt = ["格局机器人状态:" + st, "不格局机器人状态:" + st2]

							this.logArr = logsrt.concat(res.data.data.log.reverse().slice(0, 200))
						}

					},
				})



				uni.request({
					url: this.Host + "/api/v1/test/info?id=" + this.maxId, // 请求地址
					method: 'GET', // 请求方式
					success: (res) => { // 请求成功回调函数
						if (res.data.data.length > 0 && !this.isFirstLoad && this.isPlay) {

							setTimeout(() => {
								var music1 = null;
								music1 = uni.createInnerAudioContext(); //创建播放器对象
								music1.src = "../../static/new.mp3"; //选择播放的音频
								music1.play(); //执行播放	
								setTimeout(() => {
									var music2 = null;
									music2 = uni.createInnerAudioContext(); //创建播放器对象
									music2.src = "../../static/new.mp3"; //选择播放的音频
									music2.play(); //执行播放			
								}, 1000)
								setTimeout(() => {
									var music3 = null;
									music3 = uni.createInnerAudioContext(); //创建播放器对象
									music3.src = "../../static/new.mp3"; //选择播放的音频
									music3.play(); //执行播放		

								}, 2000)
							}, 1)
						}
						this.isFirstLoad = false
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

			twitterTokenSearch(it) {
				window.open("https://x.com/search?q=" + it.token)
				//window.open("https://x.com/Pipstune/status/" + it.twitter_info.tweet_id)
			},

			addressClick() {
				uni.showLoading({
					title: '加载中',
					mask: true
				})
				uni.request({
					url: this.Host + "/api/v1/test/address", // 请求地址
					method: 'POST', // 请求方式
					success: (res) => { // 请求成功回调函数
						this.walletAddresses = res.data.data.reverse()
						this.oldWalletAddresssed = this.walletAddresses.concat([])
						uni.hideLoading()
					}
				})
				this.showDialog = true

			},

			twitterClick() {
				uni.showLoading({
					title: '加载中',
					mask: true
				})
				uni.request({
					url: this.Host + "/api/v1/test/twitter", // 请求地址
					method: 'POST', // 请求方式
					success: (res) => { // 请求成功回调函数
						this.twitterList = res.data.data.reverse()
						this.oldTwitterList = this.twitterList.concat([])
						uni.hideLoading()
					}
				})
				this.showTwitterDialog = true

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
				if (it.is_twitter || it.is_search) {
					return {
						backgroundColor: "yellow"
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
			ClickNormal: function(it) {
				console.log(it)
				window.open(it)
			},
			closeDialog() {
				this.showDialog = false;
			},
			closeAnyDialog() {
				this.showAnyDialog = false;
			},
			closeAddressDialog() {
				this.showAddressDialog = false;
			},
			closeTwitterDialog() {
				this.showTwitterDialog = false;
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
			openAddTwitterModal() {
				this.editingTwitter = null;
				this.twitter_form = {
					name: '',
					isBlacklist: false,
					twitter: ''
				};
				this.showTwitterModal = true;
			},
			closeTwModal() {
				this.showTwitterModal = false;
			},
			closeModal() {
				this.showModal = false;
			},
			saveTwitter() {
				console.log(this.twitter_form)
				if (!this.twitter_form.name || !this.twitter_form.twitter) {
					uni.showToast({
						title: '请填写所有字段',
						icon: 'none'
					});
					return;
				}

				if (this.editingTwitter !== null) {
					// 编辑地址
					this.twitterList[this.editingTwitter] = {
						...this.twitter_form,
						id: this.twitterList[this.editingTwitter].id
					};
					console.log(this.form)
					uni.request({
						url: this.Host + "/api/v1/test/change_twitter", // 请求地址
						method: 'POST', // 请求方式
						header: {
							'content-type': 'application/json' // 根据后端要求设置合适的Content-Type
						},
						data: this.twitterList[this.editingTwitter],
						success: (res) => { // 请求成功回调函数
							this.twitterList = res.data.data.reverse()

							this.oldTwitterList = this.twitterList.concat([])

						}
					})
				} else {
					// 添加新地址
					const newTw = {
						...this.twitter_form
					};

					uni.request({
						url: this.Host + "/api/v1/test/add_twitter", // 请求地址
						method: 'POST', // 请求方式
						header: {
							'content-type': 'application/json' // 根据后端要求设置合适的Content-Type
						},
						data: newTw,
						success: (res) => { // 请求成功回调函数
							if (res.data.code != 1) {
								uni.showToast({
									title: res.data.msg
								})
							} else {

								this.twitterList = res.data.data.reverse()

								this.oldTwitterList = this.twitterList.concat([])

							}

						}
					})
				}

				this.closeTwModal();
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

							this.oldWalletAddresssed = this.walletAddresses.concat([])

						}
					})
				} else {
					// 添加新地址
					const newAddress = {
						...this.form
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

								this.oldWalletAddresssed = this.walletAddresses.concat([])

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

			editTwitter(index) {
				this.editingTwitter = index;
				const twitter = this.twitterList[index];
				this.twitter_form = {
					...twitter
				};
				this.showTwitterModal = true;
			},

			editAddress(index) {
				this.editingAddress = index;
				const address = this.walletAddresses[index];
				this.form = {
					...address
				};
				this.showModal = true;
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
			onCheckChange(e) {
				this.form.is_black = e.detail.value.length > 0;
			},
			onCheckTwChange(e) {
				this.twitter_form.is_black = e.detail.value.length > 0;
			},
			onCheckChange2(e) {
				this.isPlay = e.detail.value.length > 0;
			},
			isPrime(num) {
				// 小于2的数不是质数
				if (num < 2) {
					return false;
				}
				// 2是最小的质数
				if (num === 2) {
					return true;
				}
				// 排除所有偶数（除了2）
				if (num % 2 === 0) {
					return false;
				}
				// 只需检查奇数因子到sqrt(num)即可
				for (let i = 3; i <= Math.sqrt(num); i += 2) {
					if (num % i === 0) {
						return false;
					}
				}
				return true;
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
									this.oldWalletAddresssed = this.walletAddresses.concat([])

								}
							})

						}
					}
				});
			},
			detailPercent(addr) {
				if (addr.ten_token_counts == 0) {
					return "(0%)"
				}

				var p = (addr.ten_token_un_counts / addr.ten_token_counts * 100).toFixed(2)

				return "(" + p + "%)"
			},

			deleteTwitter(index) {
				uni.showModal({
					title: '确认删除',
					content: '确定删除这个推特吗？',
					success: (res) => {
						if (res.confirm) {

							uni.request({
								url: this.Host + "/api/v1/test/delete_twitter", // 请求地址
								method: 'POST', // 请求方式
								data: this.twitterList[index],
								success: (res) => { // 请求成功回调函数
									this.twitterList = res.data.data.reverse()
									this.oldTwitterList = this.twitterList.concat([])

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
		background: linear-gradient(45deg, #003366, #006699);
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
		width: 70%;
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
		height: 450px;
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

	.left-container {
		flex: 1;
		background-color: black;
		color: white;
		height: 600px;
		overflow: scroll;
		display: flex;
		flex-direction: column;
		padding: 20px;
		margin: 30px;
		border-radius: 20px;
	}

	.container-all {
		display: flex;
		flex-direction: row;
		width: 100%;
	}
</style>