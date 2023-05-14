<template>
	<view class="login-container">
		<uni-icons type="contact-filled" size="100" color="#AFAFAF"></uni-icons>
    <button type="primary" class="btn-login" open-type="getUserInfo" @getuserinfo="getUserInfo">一键登录</button>
    <text class="tips-text">登录后尽享更多权益</text>
	</view>
</template>

<script>
  import {mapMutations,mapState} from 'vuex'
  
	export default {
		data() {
			return {
				
			};
		},
    computed:{
      ...mapState('m_user',['redirectInfo'])
    },
    methods:{
      ...mapMutations('m_user',['updateUserInfo','updateToken','updateRedirectInfo']),
      
      getUserInfo(e) {
        if (e.detail.errMsg === 'getUserInfo:fail auth deny') return uni.$showMsg('您取消了登录授权！')
        
        console.log(e.detail.userInfo)
        this.updateUserInfo(e.detail.userInfo)
        
        this.getToken(e.detail)
      },
      async getToken(info){
        const [err,res] = await uni.login().catch(err => err)
        
        // if(err || res.errMsg !== 'login:ok' ) return uni.$showMsg('登录失败！')
        
        if(res.errMsg.search(/fail/g) !== -1) return uni.$showMsg('登陆失败！！')
        
        const query = {
          code:res.code,
          encryptedData:info.encryptedData,
          iv:info.iv,
          rawData:info.rawData,
          signature:info.signature
        }
        
        const {data:loginReasult} = await uni.$http.post('/api/public/v1/users/wxlogin',query)
        // if(loginReasult.meta.status !== 200 ) return uni.$showMsg('登录失败！')
        
        if(!(loginReasult.meta.status === 200 || loginReasult.meta.status === 400)) return uni.$showMsg('登录失败！！')
        
        const msg = loginReasult.message || {};
        
        if(loginReasult.meta.status === 400){
          
          for(let i=1;i<=32;i++){
            const n = Math.floor(Math.random() * 16.0).toString(16)
            msg.token +=n
          }
        }
        
        // this.updateToken(loginReasult.message.token)
        
        this.updateToken(msg.token)
        this.navigateBack()
      },
      navigateBack(){
        if(this.redirectInfo && this.redirectInfo.openType === 'switchTab') {
          uni.switchTab({
            url:this.redirectInfo.from,
            complete: () => {
              this.updateRedirectInfo(null)
            }
          })
        }
      }
    }
	}
</script>

<style lang="scss">
.login-container{
  height: 750rpx;
  background-color: #F8F8F8;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  position: relative;
  overflow: hidden;
  
  &::after{
    content: ' ';
    display: block;
    width: 100%;
    height: 40px;
    background-color: white;
    position: absolute;
    bottom: 0;
    left: 0;
    border-radius: 100%;
    transform: translateY(50%);
  }
  
  .btn-login{
    width: 90%;
    border-radius: 100px;
    margin: 15px 0;
    background-color: #C00000;
  }
  
  .tips-text{
    font-size: 12px;
    color: gray;
  }
}
</style>
