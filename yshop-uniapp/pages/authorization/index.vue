<template>
  <view class="container">
    <view v-if="!token">
      <!-- 小程序 -->
      <view v-if="$deviceType == 'routine'">
        <view class="getUserInfo">
          <text>您还未允许微信登录授权，请点击下方按钮允许微信授权登录。</text>
          <!-- <button type="primary" open-type="getUserInfo" @getuserinfo="getUserInfo">允许微信登录授权</button>
          <view style="height:20rpx"></view> -->
					<button type="primary" open-type="getPhoneNumber" @getphonenumber	="getPhoneNumber">允许微信手机授权</button>
					<view style="height:20rpx"></view>
          <button @click="back">取消微信手机授权</button>
        </view>
      </view>
      <!-- app -->
      <view v-if="$deviceType == 'app'">
        <view class="getUserInfo">
          <text>请先登录</text>
          <button type="primary" @click="this.toLogin">去登录</button>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
// import { mapState, mapMutations, mapActions } from "vuex";
import { mapState, mapGetters, mapMutations, mapActions } from "vuex";
// 组件
// import request from "@//api/request";
import { wxappAuth, getUser } from "@/api/user";
import dayjs from "dayjs";
import store from "../../store/index.js";
import cookie from "@/utils/store/cookie";
import { login, authorize } from "@/utils";

export default {
  data() {
    return {
      authorize: false,
			// code: null
    };
  },
  computed: {
    ...mapState(["isAuthorization", "$deviceType", "token"])
  },
	onLaunch() {
	},
  onShow() {
    // // 先校验用户是否授权，如果没有授权，显示授权按钮
		// let that = this;
		// uni.login({
		// 	provider: 'weixin',
		// 	success: function(loginRes){
		// 		console.log(loginRes);
		// 		that.code = loginRes.code;
		// 		console.log('authorization登录凭证为: ',that.code);
		// 	}
		// })	
  },
  onHide() {
    this.UPDATE_AUTHORIZATIONPAGE(false);
    this.changeAuthorization(false);
  },
  onUnload() {
    this.UPDATE_AUTHORIZATIONPAGE(false);
    this.changeAuthorization(false);
  },
  methods: {
    ...mapActions(["changeAuthorization", "setUserInfo"]),
    ...mapMutations(["UPDATE_AUTHORIZATIONPAGE", "CHANGE_TABTAR"]),
    
    back() {
      this.$yrouter.switchTab({
        path: "/pages/home/index",
        query: {}
      });
    },
		
    getUserInfo(data) {
      if (data.detail.errMsg == "getUserInfo:fail auth deny") {
        uni.showToast({
          title: "取消授权",
          icon: "none",
          duration: 2000
        });
        return;
      }
      uni.showLoading({
        title: "登录中"
      });
      login()
        .then(res => {
          this.$yrouter.replace({ path: cookie.get("redirect") });
        })
        .catch(error => {
          console.log(error);
          uni.showToast({
            title: error,
            icon: "none",
            duration: 2000
          });
        });
    },
		
		getPhoneNumber(e){
			let that = this;
			console.log(e);
			if(e.detail.errMsg == "getPhoneNumber:ok"){
				uni.showLoading({
				  title: "绑定中"
				});
			}
			//获取当前环境的服务商
			uni.getProvider({
				service: "oauth",
				success: function(res){
					console.log(res);
					// 此处可以排除h5
					if(res.provider){
						uni.login({
							// provider: 'weixin',
							success: function(loginRes){
								console.log(loginRes);
								console.log('authorization登录凭证为: ',loginRes.code);
								let data = {
									code: loginRes.code,
									encryptedData: e.detail.encryptedData,
									iv: e.detail.iv
								}
								wxappAuth(data).then(res=>{
									console.log(res);
									console.log('token为: ',res.data.token)
									console.log(1)
									if(res.status == 200){
										that.$yrouter.switchTab({
											path: "/pages/home/index",
											query: {}
										});
									}
									store.commit("LOGIN", res.data.token, dayjs(res.data.expires_time));
									getUser().then(user => {
										console.log('获取用户信息成功')
										store.dispatch('setUserInfo', user.data)
										resolve(user)
									}).catch(error => {
										console.log('获取用户信息失败');
									});
								})
								// uni.request({
								// 	url: 'http://192.168.1.4:8010/api/wxapp/auth',
								// 	data: data,
								// 	method: 'POST',
								// 	header: {
								// 		'Content-Type': 'application/json'
								// 	},
								// 	success:function(res){
								// 		console.log(res);
								// 		console.log('token为: ',res.data.data.token)
								// 		console.log(1)
								// 		if(res.errMsg == "request:ok"){
								// 			that.$yrouter.switchTab({
								// 			  path: "/pages/home/index",
								// 			  query: {}
								// 			});
								// 		}
								// 		store.commit("LOGIN", res.data.data.token, dayjs(res.data.data.expires_time));
								// 		getUser().then(user => {
								// 			console.log('获取用户信息成功')
								// 			store.dispatch('setUserInfo', user.data)
								// 			resolve(user)
								// 		}).catch(error => {
								// 			console.log('获取用户信息失败');
								// 		});
								// 	}
								// })
							}
						})
					}
				}
			})
			
		},
		
    toLogin() {
      this.$yrouter.push({
        path: "/pages/user/Login/index",
        query: {}
      });
    }
  },
  mounted() {
    
  }
};
</script>

<style lang="less">
.sp-cell {
  height: 20rpx;
}

.getUserInfo {
  display: flex;
  align-items: center;
  flex-direction: column;
  padding: 30px;

  text {
    font-size: 30rpx;
    text-align: center;
    margin-bottom: 20px;
  }
}

.container {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  position: relative;
}

.tab-bar {
  font-size: 0;
  display: flex;
  align-items: center;
  background: rgba(255, 255, 255, 0.9);
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  z-index: 99;
  border-top: 1px solid rgba(248, 248, 248, 1);

  .tab-bar-item {
    flex: 1;
    height: 49px;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;

    &.active {
      text {
        color: #ee7559;
      }

      .tab-bar-pic {
        display: none;
        background: #f9f9f9;

        &.active {
          display: block;
        }
      }
    }

    .tab-bar-pic {
      display: block;
      background: #f9f9f9;

      &.active {
        display: none;
      }
    }
  }

  .tab-bar-pic {
    width: 25px;
    height: 25px;
    background: #f9f9f9;

    image {
      width: 25px;
      height: 25px;
    }
  }

  .tab-bar-pic-active {
  }

  text {
    font-size: 10px;
    color: rgb(160, 160, 160);
    line-height: 10px;
    margin-top: 5px;
  }
}

.tab-bar-bg {
  padding-top: 46px;
  width: 100%;
}

.view-item {
  display: none;
  width: 100%;
}

.view-item-active {
  display: block;
}

.getUserInfo {
  display: flex;
  align-items: center;
  flex-direction: column;
  padding: 30px;

  p {
    margin-bottom: 20px;
  }
}

._van-dialog {
  z-index: 99999999999;
}
</style>
