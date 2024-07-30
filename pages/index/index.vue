<template>
	<view>
		<!-- 导航 -->
		<!-- 	<navBar></navBar>
 -->

		<view style="font-size: 18px;font-weight: bold">店铺列表</view>
		<view class="box" @click="onStore(item)" v-for="(item,index) in list " :key="index">
			<view>店铺姓名{{item.name}}</view>
			<view>店铺状态{{item.status}}</view>
			<view>店铺介绍{{item.distort}}</view>
		</view>


	</view>
</template>
<script>
	// import navBar from "../common/Navbar/navbar.vue" //标题
	import {
		mapState,
		mapMutations
	} from "vuex"
	import {
		getStoreList,
		userLogin
	} from '../../pages/api/api.js'
	export default {


		data() {
			return {
				list: []
			}
		},

		onLoad() {
			this.getData()
		},
		onShow() {
			this.init()
		},
		methods: {
			// 从vuex信息
			...mapState([
				"shopInfo", //店铺信息
				"shopPhone", //电话
				"orderListData",
				"baseUserInfo", //用户信息
				"lodding",
				"token", //token
				"deliveryFee", //配送费
			]),
			//   vuex储存信息
			...mapMutations([
				"setShopInfo", //设置店铺信息
				"setShopPhone", //设置电话
				"setShopStatus", //设置店铺状态
				"initdishListMut", //设置购物车订单
				"setStoreInfo",
				"setBaseUserInfo", //设置用户基本信息
				"setLodding",
				"setToken", //设置token
				"setDeliveryFee", //设置配送费
			]),
			init() {
				getStoreList()
					.then((success) => {
						if (success.code === 1) {
							this.list = success.data
						}
					})
					.catch((err) => {})
			},
			onStore(item) {

				uni.navigateTo({
					url: `/pages/index/indexdetile?name=${item.name}&status=${item.status},`
				})

				uni.setStorageSync("storeId", item.id)
			},
			getData() {
				let res = wx.getMenuButtonBoundingClientRect()
				let _this = this
				// 获取店铺状态
				// this.getShopInfo()
				this.selectHeight = res.height
				if (this.token() === "") {
					uni.showModal({
						title: "温馨提示",
						content: "亲，授权微信登录后才能点餐！",
						showCancel: false,
						success(res) {
							if (res.confirm) {
								let jsCode = ""
								uni.login({
									provider: "weixin",
									success: (loginRes) => {
										if (loginRes.errMsg === "login:ok") {
											jsCode = loginRes.code
										}
									},
								})
								// 授权
								uni.getUserProfile({
									desc: "登录",
									success: function(userInfo) {
										_this.setBaseUserInfo(userInfo.userInfo)
										const params = {
											code: jsCode,
											// 传递地理位置信息
										}
										// 获取定位信息
										uni.getLocation({
											type: 'gcj02',
											isHighAccuracy: true
										}).then(([err, result]) => {
											if (err) {
												uni.showToast({
													title: "获取地理位置失败",
													icon: "none"
												})
											} else {
												if (process.env.NODE_ENV === '"development"') {
													params.location =
														`116.481488,39.990464` //	先写死在北京
												} else {
													params.location =
														`${result.longitude},${result.latitude}`
												}

												userLogin(params)
													.then((success) => {
														if (success.code === 1) {
															_this.setToken(success.data
																.token)
															// 保存配送费
															// _this.setDeliveryFee(success
															// 	.data.deliveryFee)
															// // 保存商铺信息
															// _this.setShopInfo({
															// 	shopName: success
															// 		.data.shopName,
															// 	shopAddress: success
															// 		.data
															// 		.shopAddress,
															// 	shopId: success
															// 		.data.shopId,
															// })
															_this.init()
														}
													})
													.catch((err) => {})



											}

										})

									},
									fail: function(err) {},
								})
							}
						},
					})
				}
			},

		}
	}
</script>

<style scoped>
	/* #endif */
	.box {
		margin: 20px 0;
	}
</style>