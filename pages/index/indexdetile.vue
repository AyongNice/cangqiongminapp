<template>
	<view>
		<!-- 导航 -->
		<navBar></navBar>
		<!-- end -->
		<view class="home_content" :style="{ paddingTop: ht + 'px' }" @touchmove.stop.prevent="disabledScroll">
			<!-- 店铺基本信息 -->
			<view class="restaurant_info_box">
				<view class="restaurant_info">
					<!-- 上部 -->
					<view class="info_top">
						<view class="info_top_left">
							<image class="logo_ruiji" src="../../static/logo_ruiji.png"></image>
						</view>
						<view class="info_top_right">
							<view class="right_title">
								<text>{{name}}</text>
								<view class="businessStatus" v-if="shopStatus === 1">营业中</view>
								<view class="businessStatus close" v-else>休息中</view>
							</view>
							<view class="right_details">
								<!-- 中 -->
								<view class="details_flex">
									<image class="top_icon" src="../../static/money.png"></image>
									<text class="icon_text">配送费{{ deliveryFee() }}元</text>
								</view>
							</view>
						</view>
					</view>
					<!-- 下部---信息简介 -->
					<view class="info_bottom">
						<view>
							<view class="word">苍穹餐厅为顾客打造专业的大众化美食外送餐饮</view>
							<view class="address">
								<icon></icon>
								{{ shopInfo().shopAddress || "商家店铺获取中.." }}
							</view>
						</view>
						<view>
							<view class="phone" @click="handlePhone('bottom')">
								<icon class="phoneIcon"></icon>
							</view>
						</view>
					</view>
				</view>
			</view>
			<!-- end -->
			<!-- 菜单列表 -->
			<view class="restaurant_menu_list" v-if="shopStatus === 1">
				<view class="type_list">
					<scroll-view scroll-y scroll-with-animation class="u-tab-view menu-scroll-view"
						:scroll-top="scrollTop + 100" :scroll-into-view="itemId">
						<view class="type_item" id="target" :class="[typeIndex == index ? 'active' : '']"
							v-for="(item, index) in typeListData" :key="index" @tap.stop="swichMenu(item, index)">
							<view class="item" :class="item.name.length > 5 ? 'allLine' : ''">{{ item.name }}</view>
						</view>
						<view class="seize_seat"></view>
					</scroll-view>
				</view>
				<scroll-view class="vegetable_order_list" scroll-y="true" scroll-top="0rpx"
					v-if="dishListItems && dishListItems.length > 0">
					<view class="type_item" v-for="(item, index) in dishListItems" :key="index">
						<!-- 点击查看详情 -->
						<view class="dish_img" @click="openDetailHandle(item)">
							<image mode="aspectFill" :src="item.image" class="dish_img_url"></image>
						</view>
						<view class="dish_info">
							<view class="dish_name" @click="openDetailHandle(item)">{{
                item.name
              }}</view>
							<view class="dish_label" @click="openDetailHandle(item)">{{
                item.description || item.name
              }}</view>
							<view class="dish_label" @click="openDetailHandle(item)">月销量0</view>
							<view class="dish_price">
								<text class="ico">￥</text>
								{{ item.price.toFixed(2) }}
							</view>
							<view class="dish_active" v-if="!item.flavors || item.flavors.length === 0">
								<!-- 减菜 -->
								<image v-if="item.dishNumber >= 1" src="../../static/btn_red.png"
									@click="redDishAction(item, '普通')" class="dish_red"></image>
								<text v-if="item.dishNumber > 0" class="dish_number">{{
                  item.dishNumber
                }}</text>
								<!-- 加菜 -->
								<image src="../../static/btn_add.png" class="dish_add"
									@click="addDishAction(item, '普通')"></image>
							</view>
							<view class="dish_active_btn" v-else>
								<view class="check_but" @click="moreNormDataesHandle(item)">选择规格</view>
							</view>
						</view>
					</view>
					<view class="seize_seat"></view>
				</scroll-view>
				<view class="no_dish" v-else>
					<view v-if="typeListData.length > 0">该分类下暂无菜品</view>
				</view>
			</view>
			<view class="restaurant_close">店铺已打烊</view>
			<!-- end -->
			<view class="mask-box"></view>
			<!-- 底部去结算 -->
			<!-- 购物车里没有订单的状态 -->
			<view class="footer_order_buttom" v-if="orderListData().length === 0 || shopStatus !== 1">
				<view class="order_number">
					<image src="../../static/btn_waiter_nor.png" class="order_number_icon" mode=""></image>
				</view>
				<view class="order_price">
					<text class="ico">￥</text>
					0
				</view>
				<view class="ord<strong></strong>er_but">去结算</view>
			</view>
			<!-- end -->
			<!-- 购物车里有订单结算 -->
			<view class="footer_order_buttom order_form" v-else>
				<view class="orderCar" @click="() => (openOrderCartList = !openOrderCartList)">
					<view class="order_number">
						<image src="../../static/btn_waiter_sel.png" class="order_number_icon" mode=""></image>
						<view class="order_dish_num">{{ orderDishNumber }}</view>
					</view>
					<view class="order_price">
						<text class="ico">￥</text>
						{{ orderDishPrice.toFixed(2) }}
					</view>
				</view>
				<view class="order_but" @click="goOrder()">去结算</view>
			</view>
			<!-- end -->
			<!-- 选择多规格弹层 - start -->
			<view class="pop_mask" v-show="openMoreNormPop">
				<popMask :moreNormDishdata="moreNormDishdata" :moreNormdata="moreNormdata" :flavorDataes="flavorDataes"
					@checkMoreNormPop="checkMoreNormPop" @addShop="addShop" @closeMoreNorm="closeMoreNorm"></popMask>
			</view>
			<!-- 选择多规格 - end -->
			<!-- 菜品详情弹层 - start -->
			<!-- openDetailHandle 这个函数触发的菜品详情 -->
			<view class="pop_mask" v-show="openDetailPop" style="z-index: 9999">
				<dishDetail :dishDetailes="dishDetailes" :openDetailPop="openDetailPop" :dishMealData="dishMealData"
					@redDishAction="redDishAction" @addDishAction="addDishAction"
					@moreNormDataesHandle="moreNormDataesHandle" @dishClose="dishClose"></dishDetail>
			</view>
			<!-- 菜品详情 - end -->
			<!-- 购物车弹框 - start -->
			<view class="pop_mask" v-if="openOrderCartList" @click="openOrderCartList = !openOrderCartList">
				购物车弹框 - start
				<view class="cart_pop" @click.stop="openOrderCartList = openOrderCartList">
					<view class="top_title">
						<view class="tit">购物车</view>
						<view class="clear" @click.stop="clearCardOrder()">
							<image class="clear_icon" src="../../static/clear.png" mode=""></image>
							<text class="clear-des">清空</text>
						</view>
					</view>
					<scroll-view class="card_order_list" scroll-y="true" scroll-top="40rpx">

						<view class="type_item_cont" v-for="(item, ind) in orderAndUserInfo" :key="ind">
							<view class="type_item" v-for="(obj, index) in item.dishList" :key="index">
								<view class="dish_img">
									<image mode="aspectFill" v-if="obj.image" :src="obj.image" class="dish_img_url">
									</image>
								</view>
								<view class="dish_info">
									<view class="dish_name">{{ obj.name }}</view>
									<view class="dish_dishFlavor" v-if="obj.dishFlavor">{{
				          obj.dishFlavor
				        }}</view>
									<view class="dish_price">
										<text class="ico">￥</text>
										{{ obj.amount }}
									</view>
									<view class="dish_active">
										<image v-if="obj.number && obj.number > 0" src="../../static/btn_red.png"
											@click.stop="redDishAction(obj, '购物车')" class="dish_red" mode=""></image>
										<text v-if="obj.number && obj.number > 0" class="dish_number">{{
				            obj.number
				          }}</text>
										<image src="../../static/btn_add.png" class="dish_add"
											@click.stop="addDishAction(obj, '购物车')" mode=""></image>
									</view>
								</view>
							</view>
						</view>
						<view class="seize_seat"></view>
					</scroll-view>
				</view>
			</view>
			<!-- 购物车弹框 - end -->
			<view class="pop_mask" v-show="loaddingSt">
				<view class="lodding">
					<image class="lodding_ico" src="../../static/lodding.gif" mode=""></image>
				</view>
			</view>
			<!-- 电话弹层 -->
			<phone ref="phone" :phoneData="phoneData" @closePopup="closePopup"></phone>
			<!-- end -->
			<!-- 店面打烊弹层 -->
			<view class="colseShop" v-if="shopStatus === 0">
				<view class="shop">本店已打样</view>
			</view>
			<!-- end -->
		</view>
	</view>
</template>

<script>
	import myMixin from "./index.js";
	import popCart from "./components/popCart.vue" //购物车弹出层
	export default {
		mixins: [myMixin],
		components: {
			popCart
		}
	}
</script>

<style src="./style.scss" lang="scss" scoped></style>
<style scoped>
	/* #ifdef MP-WEIXIN || APP-PLUS */
	::v-deep ::-webkit-scrollbar {
		display: none !important;
		width: 0 !important;
		height: 0 !important;
		-webkit-appearance: none;
		background: transparent;
		color: transparent;
	}

	/* #endif */
</style>
<style lang="scss" scoped>
	.cart_pop {
		width: 100%;
		position: absolute;
		bottom: 0;
		left: 0;
		height: 60vh;
		background-color: #fff;
		border-radius: 8rpx 8rpx 0 0;
		padding: 20rpx 30rpx 30rpx 30rpx;
		box-sizing: border-box;

		.top_title {
			display: flex;
			justify-content: space-between;
			border-bottom: solid 1px #ebeef5;
			padding-bottom: 20rpx;

			.tit {
				font-size: 40rpx;
				font-weight: bold;
				color: #20232a;
			}

			.clear {
				color: #999999;
				font-size: 28rpx;
				font-weight: 400;
				display: flex;
				align-items: center;
				font-family: PingFangSC, PingFangSC-Regular;

				// position: relative;
				// top: 14rpx;
				.clear_icon {
					// position: relative;
					// top: 0rpx;
					width: 30rpx;
					height: 30rpx;
					margin-right: 8rpx;
				}

				.clear-des {
					height: 56rpx;
					line-height: 56rpx;
				}
			}
		}

		.card_order_list {
			background-color: #fff;
			padding-top: 40rpx;
			box-sizing: border-box;
			height: calc(100% - 0rpx);
			flex: 1;
			position: relative;

			.type_item_cont {
				.user_info {
					display: flex;
					margin-bottom: 20rpx;

					.user_avatar {
						.user_avatar_icon {
							width: 42rpx;
							height: 42rpx;
							border-radius: 42rpx;
						}

						margin-right: 20rpx;
					}

					.user_name {
						color: #19232b;
						font-size: 24rpx;
					}
				}
			}

			.type_item {
				display: flex;
				margin-bottom: 40rpx;

				.dish_img {
					width: 128rpx;
					margin-right: 30rpx;

					.dish_img_url {
						display: block;
						width: 128rpx;
						height: 128rpx;
						border-radius: 8rpx;
					}
				}

				.dish_info {
					position: relative;
					flex: 1;
					padding-bottom: 120rpx;
					border-bottom: solid 1px #ebeef5;

					.dish_name {
						font-size: 32rpx;
						line-height: 40rpx;
						color: #333333;
						font-family: PingFangSC, PingFangSC-Semibold;
						font-weight: 600;
					}

					.dish_price {
						font-size: 32rpx;
						color: #e94e3c;
						position: absolute;
						bottom: 24rpx;

						.ico {
							font-size: 24rpx;
						}
					}

					.dish_active {
						position: absolute;
						right: 20rpx;
						bottom: 20rpx;
						display: flex;

						.dish_add,
						.dish_red {
							display: block;
							width: 72rpx;
							height: 72rpx;
						}

						.dish_number {
							padding: 0 10rpx;
							line-height: 72rpx;
							font-size: 30rpx;
							font-family: PingFangSC, PingFangSC-Medium;
							font-weight: 500;
						}
					}
				}
			}

			&::before {
				content: "";
				position: absolute;
				width: 100vw;
				height: 120rpx;
				z-index: 99;
				background: linear-gradient(0deg,
						rgba(255, 255, 255, 1) 10%,
						rgba(255, 255, 255, 0));
				bottom: 0px;
				left: 0px;
			}

			.seize_seat {
				width: 100%;
				height: 120rpx;
			}
		}

		.dish_dishFlavor {
			position: absolute;
			left: 0;
			top: 40rpx;
		}
	}
</style>