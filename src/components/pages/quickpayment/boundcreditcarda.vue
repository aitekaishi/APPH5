<template>
	<div style="height: 100vh;width: 100vw;overflow: hidden;" class="two">
		<loading :show="loading"></loading>
    	<toast v-model="errorStatus" type="text" time=3000>{{errorMsg}}</toast>
		<div class="index">
			<x-header :left-options="{showBack: true,backText: '',preventGoBack:true}" @on-click-back="back">绑定信用卡</x-header>
		</div>
		<div class="mycard">请绑定商户本人你的信用卡</div>
		<div class="card">
			<group>
				<x-input class='assa' title="持卡人  " v-model="username" readonly=true></x-input>
				<x-input title="卡号   " placeholder="请输入信用卡卡号" mask="9999 9999 9999 9999 9999 9999" v-model="cardnum" max=27></x-input>
			</group>
		</div>
    	<div class="nextcard">
    		<x-button class="XB" :disabled="XB" @click.native="next()"  type="warn">下一步</x-button>
    	</div>
    	<div style="height: 200px;"></div>
 </div>
</template>

<script>
	//import store from '@/store'
	import {XButton,XHeader,XInput,Group,Toast,Loading} from 'vux'
	import api from '../../../components/other/api-url'
  	import globalPM from '../../../components/other/global-parameter'
 	import axios from 'axios'
  	import Cookies from 'js-cookie'
	import router from '../../../router/index'
	import common from '../../../components/other/common'
	export default {
		components: {XButton,XHeader,XInput,Group,Toast,Loading},
		data (){ 
			return {
				XB:true,
				username:"",
				cardnum:'',
				loading:false,
				errorStatus: false,
        		errorMsg: '',
        		type:"warn",
			}
		},
		methods:{
			back(){
				router.push('/cardlist')
			},
			/*验证信用卡是否支持   
			/user/payment/validCardSupport   
			传入参数 参数类型 必需 描述
			cardNo String Y 信用卡号
			传出参数   
			cardId String Y 身份证号
			bankCode String Y 银行简码
			bankNum String Y 银行代码
			bankName String Y 银行名称*/
			next:function  () {
				const _this = this
				var x = _this.cardnum.replace(/\s+/g,"");
				localStorage.setItem("dddddd",_this.cardnum);
	      		axios.post(api + '/user/repay/getCreditCardInfo?&cardNo=' + x+'&channelId=' + localStorage.getItem("xianid") + '&uid=' + localStorage.getItem('uid') + '&token=' + localStorage.getItem('token') + '&valid=0' + globalPM).then(function (res) {
					console.log(res.data)
			        if (res.data.code === 0) {
			            localStorage.setItem("creditcardname",res.data.data.bankName+'-'+res.data.data.bankCode)//支付卡银行名称
			            localStorage.setItem("creditcardnum",x)//信用卡卡号
			            localStorage.setItem("creditcardcode",res.data.data.bankCode)//支付卡银行简码
			            localStorage.setItem("creditcardtype",res.data.data.cardTypeDesc)//支付卡种类
			            localStorage.setItem("cardId",res.data.data.cardId)
			            localStorage.setItem("subUpdate",res.data.data.subUpdate)
			            /*_this.errorMsg = "下一步"
			        	_this.errorStatus = true*/
			            setTimeout(function  () {
			            	router.push('/cardinfoa');
			            },500)
			        }else{
			        	_this.errorMsg = res.data.msg
			        	_this.errorStatus = true
			        }
			        if (res.data.code === 110) {
			            router.push('/login')
			        }
				})
			}
		},
		created(){
			const _this = this;
			console.log("进入boundcreditcard")
			if (localStorage.getItem('dddddd') !=null) {
				_this.XB = false;
			}
			_this.cardnum = localStorage.getItem('dddddd')
			localStorage.removeItem('dddddd')
			console.log(localStorage.getItem('name'))
			axios.post(api + '/user/real/info?'+'&uid=' + localStorage.getItem('uid') + '&token=' + localStorage.getItem('token') + '&valid=0' + globalPM).then(function (res) {
				console.log(res.data)
		        if (res.data.code === 0) {
		            _this.username = res.data.data.realName
		        }else{
		        	_this.errorMsg = res.data.msg
		        	_this.errorStatus = true
		        }
		        if (res.data.code === 110) {
		            router.push('/login')
		        }
			})
		},
		updated(){
			//判断提交按钮可以点击
			const _this = this
			if (_this.cardnum !== ''&&_this.username !== '') {
				_this.XB = false;
			}else{
				_this.XB=true;
			}
		},
	}
</script>
<style lang="less">
	@import url("less/x-header.less");
	@import url("../../../assets/main.less");
	@import url("less/default.less");
	@import url("less/button.less");
	@import url("../../../assets/update.less");
	.mycard{
		margin: 0.5rem 0 -0.7rem 1rem;
		color: #9B9B9B;
		white-space:nowrap;
	}
	.nextcard{
		position: relative;
		z-index: 10;
		margin-top: 2rem;
		text-align: center;
		.XB{
			width: 90%;
		}
	}
	.card{
		.weui-label{
			width: 4em !important;
		}
		label{
			color: #000;
		}
		input{
			color: #000;
		}
		.vux-x-input.disabled .weui-input{
			    -webkit-text-fill-color: #000;
		}
	}
	
</style>
				