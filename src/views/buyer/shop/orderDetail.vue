<template>
    <div class="m-orderDetail">
      <div class="m-orderDetail-status" v-if="order_info.omstatus == 0">
        <div>
          <span>买家待付款</span>
          <span class="duration-time" v-if="order_info.duration">{{order_info.min}}:{{order_info.sec}}</span>
          <!-- <span class="m-icon-order-status m-pay" ></span> -->
        </div>
      </div>
      <div class="m-orderDetail-status" v-if="order_info.omstatus == -40">
        <span >买家已取消</span>
        <!-- <span class="m-icon-order-status m-pay" ></span> -->
      </div>
      <div class="m-orderDetail-status" v-if="order_info.omstatus == 10">
        <span>卖家待发货</span>
        <!-- <span class="m-icon-order-status m-pay" ></span> -->
      </div>
      <div class="m-orderDetail-status" v-if="order_info.omstatus == 20">
        <span >买家待收货</span>
        <!-- <span class="m-icon-order-status m-send" ></span> -->
      </div>
      <div class="m-orderDetail-status" v-if="order_info.omstatus == 30 || order_info.omstatus == 25 || order_info.omstatus == 26">
        <span>买家待评价</span>
        <!-- <span class="m-icon-order-status m-send"></span> -->
      </div>
      <div class="m-order-one-part">
        <div class="m-user-text">
          <span class="m-icon-loc"></span>
          <div>
            <p>
              <span>{{order_info.omrecvname}}</span>
              <span class="m-user-tel">{{order_info.omrecvphone}}</span>
            </p>
            <p class="m-bottom-text">
              收货地址：{{order_info.omrecvaddress}}
            </p>
          </div>
        </div>
      </div>
      <div class="m-order-one-part" v-if="logistic_info && order_info.omlogistictype != 10">
        <div class="m-user-text" @click="changeRoute('/logisticsInformation')">
          <span class="m-icon-wuliu m-done"></span>
          <div class="m-flex-between">
            <div>
              <p class="m-wuliu-text">{{logistic_info.ollastresult.status}}</p>
              <p class="w-wuliu-time">{{logistic_info.ollastresult.time}}</p>
            </div>
            <span class="m-icon-more"></span>
          </div>
        </div>
      </div>
      <div class="m-order-one-part">
        <div class="m-order-store-tile">
          <div @click.stop="changeRoute('/brandDetail')">
            <span class="m-icon-store"></span>
            <span class="m-store-name">{{order_info.pbname}}</span>
            <span class="m-icon-more"></span>
          </div>
          <span class="m-red" v-if="from == 'activityProduct'">
            <span v-if="order_info.omstatus == 25">已完成</span>
            <span class="m-red" v-else>{{order_info.omstatus_zh}}</span>
          </span>
          <span v-else>
            <span class="m-red" v-if="refund">{{refund.orastate_zh}}</span>
            <span class="m-red" v-else>{{order_info.omstatus_zh}}</span>
          </span>
        </div>
        <div class="m-product-box" v-for="(item, index) in order_info.order_part">
          <div class="m-order-product-ul">
            <div class="m-product-info" @click.stop="changeRoute('/productDetail',item)">
              <img :src="item.prmainpic" class="m-product-img" alt="">
              <div class="m-product-info-text">
                <p class="m-flex-between">
                <span class="m-product-name">{{item.prtitle}}</span>
                  <span v-if="item.tlsprice">￥{{item.tlsprice}}</span>
                  <span v-else>￥{{item.skuprice}}</span>
                </p>
                <p class="m-flex-between m-sku-text m-ft-22">
                  <span class="m-product-label">
                    <template v-for="(key,k) in item.skuattritedetail" >
                        <span >{{key}}</span>
                        <span v-if="k < item.skuattritedetail.length-1">；</span>
                      </template>
                  </span>
                  <span >x{{item.opnum}}</span>
                </p>
              </div>
            </div>
          </div>
          <!-- <p class="m-flex-between m-ft-22">
            <span>运费</span>
            <span>￥{{order_info.omfreight | money}}</span>
          </p> -->
          <p class="m-flex-between m-ft-22">
            <span>实付款（含运费）</span>
            <span class="w-price">￥{{item.opsubtotal | money}}</span>
          </p>
          <p class="m-back-btn" v-if="from !== 'afterSales'  && !order_info.ominrefund">
            <span class="active" @click="changeRoute('/selectBack', item)" v-if="(order_info.omstatus == 10 || order_info.omstatus == 20 || order_info.omstatus == 25 || order_info.omstatus == 26) && !item.order_refund_apply && order_info.omfrom != 80"">发起退款</span>
            <span @click="changeRoute('/backDetail', item)" v-if="(order_info.omstatus == 10 || order_info.omstatus == 20 || order_info.omstatus == 25 || order_info.omstatus == 26) && item.order_refund_apply">查看退款</span>
            <span @click="changeRoute('/storekeeper/IDCardApprove')" v-if="order_info.omlogistictype == 10 && order_info.omstatus == 30 && from !== 'activityProduct'">身份认证</span>
          </p>
        </div>
        <div class="m-total-money">
          共{{totalProductNum}}件商品 合计：<span class="w-price" v-if="order_info.omfrom == 80">{{order_info.omtruemount}}星币</span><span class="w-price" v-else>￥{{order_info.omtruemount | money}}</span>（含运费{{order_info.omfreight | money}}）
          <p class="m-back-btn">
            <span class="active" @click="changeRoute('/selectBack', item)" v-if="(order_info.omstatus == 10 || order_info.omstatus == 20 || order_info.omstatus == 25 || order_info.omstatus == 26) && order_info.omfrom != 80">全部退款</span>
          </p>
        </div>
      </div>
      <div class="m-order-one-part">
        <p>
          <!-- <span class="m-border"></span> -->
          <span>订单信息</span>
        </p>
        <div class="m-ft-22 m-time-text">
          <p><span class="w-ft-text">订单编号：</span> {{order_info.omno}}</p>
          <p><span class="w-ft-text">创建时间：</span> {{order_info.createtime}}</p>
          <p v-if="order_info.pay_time"><span class="w-ft-text">付款时间：</span> {{order_info.pay_time}}</p>
          <p v-if="order_info.send_time"><span class="w-ft-text">发货时间：</span> {{order_info.send_time}}</p>
        </div>
      </div>
      <div class="m-order-one-part" v-if="refund">
        <p>
          <!-- <span class="m-border"></span> -->
          <span>售后信息</span>
        </p>
        <div class="m-ft-22 m-time-text">
          <p><span class="w-ft-text">订单状态：</span>{{refund.orastate_zh}} {{refund.orastatus_zh}}</p>
          <p><span class="w-ft-text">退款金额：</span>￥{{refund.oramount | money}}</p>
          <p><span class="w-ft-text">申请理由：</span>{{refund.orareason}}</p>
          <p><span class="w-ft-text">货物状态：</span>{{refund.oraproductstatus_zh}}</p>
          <p v-if="refund.oraaddtion"><span class="w-ft-text">附加留言：</span>{{refund.oraaddtion}}</p>
          <p v-if="refund.orachecktime"><span class="w-ft-text">处理时间：</span>{{refund.orachecktime}}</p>
          <!--<p v-if="refund.oracheckreason">审核回复：{{refund.oracheckreason}}</p>-->
          <p><span class="w-ft-text">申请时间：</span>{{refund.createtime}}</p>
          <p><span class="w-ft-text">售后单号：</span>{{refund.orasn}}</p>
        </div>
      </div>
      <div class="m-order-one-part" v-if="refund_notes">
        <p>
          <!-- <span class="m-border"></span> -->
          <span>售后审核信息</span>
        </p>
        <div class="m-ft-22 m-time-text">
          <p>审核回复：{{refund_notes.ornabo}}</p>
        </div>
      </div>

      <div class="m-align-right" v-if="from !== 'activityProduct' && from !== 'afterSales' && !order_info.ominrefund">
        <span class="w-footer-1" v-if="order_info.omstatus == -40" @click="deleteOrder">删除订单</span>
        <span class="w-footer-2" v-if="order_info.omstatus == 0" @click="cancelOrder">取消订单</span>
        <span class="w-footer-1" v-if="(order_info.omstatus == 10) && !part_refund" @click="changeRoute('/selectBack', 'order')">联系卖家</span>
        <!-- <span class="" @click="changeRoute('/logisticsInformation')" v-if="order_info.omstatus==20">查看物流</span> -->
        <span class="w-footer-2 active" v-if="order_info.omstatus == 0" @click="payBtn">立即付款</span>
        <span class="w-footer-1 active" v-if="order_info.omstatus == 20" @click="orderConfirm">确认收货</span>
        <span class="w-footer-1 active" v-if="order_info.omstatus == 25" @click="changeRoute('/addComment')">立即评价</span>
      </div>
      <!--活动订单评价-->
      <div class="m-align-right" v-if="from == 'activityProduct'">
        <span class="active" v-if="order_info.omstatus == 25" @click="changeRoute('/addComment')">评价</span>
      </div>
      <!-- <bottom></bottom> -->
      <div class="m-modal-pwd" v-if="show_modal ">
        <div class="m-modal-state" @click.self="show_modal = false">
          <div class="m-one">
            <img src="/static/images/product/icon-close.png" class="m-close" @click="show_modal = false" alt="">
            <h3>请输入星币支付密码</h3>
          </div>
          <div class="m-one">
            <img src="/static/images/newpersonal/icon-star-can.png" class="m-icon" alt="">
            <span class="m-star-num">{{omtruemount}}</span>
          </div>
          <div class="m-one m-flex-between">
            <span>星币余额</span>
            <span >{{usintegral}}币</span>
          </div>
          <div >
            <input ref="pwd" type="tel" maxlength="6" v-model="msg" class="pwd" unselectable="on" autofocus />
            <ul class="m-input-box" @click="focus">
              <li :class="msg.length == 0?'psd-blink':''" class="m-setPwd-input"><i v-if="msg.length > 0"></i><s></s></li>
              <li :class="msg.length == 1?'psd-blink':''" class="m-setPwd-input"><i v-if="msg.length > 1"></i><s></s></li>
              <li :class="msg.length == 2?'psd-blink':''" class="m-setPwd-input"><i v-if="msg.length > 2"></i><s></s></li>
              <li :class="msg.length == 3?'psd-blink':''" class="m-setPwd-input"><i v-if="msg.length > 3"></i><s></s></li>
              <li :class="msg.length == 4?'psd-blink':''" class="m-setPwd-input"><i v-if="msg.length > 4"></i><s></s></li>
              <li :class="msg.length == 5?'psd-blink':''" class="m-setPwd-input"><i v-if="msg.length > 5"></i><s></s></li>
            </ul>
            <p class="m-forget" @click="changeRoute('/personal/editInput', 'forget')">忘记密码？</p>
          </div>
        </div>
      </div>
    </div>
</template>

<script>
  import common from '../../../common/js/common';
  import bottom from '../components/bottomService';
  import axios from 'axios';
  import api from '../../../api/api';
  import { Toast, MessageBox } from 'mint-ui';

  export default {
    data() {
      return {
        order_info: { omtruemount: '', omfreight: '' },
        // order_info: { omtruemount: '', omfreight: '', min: 0, sec: 0 },
        logistic_info: null,
        from: "",
        refund: null,
        refund_notes: null,
        part_refund: false,
        show_modal:false,
        msg:'',
        omtruemount:0,
        omid:'',
        usintegral:0
      }
    },
    components: { bottom },
    inject: ['reload'],
    mounted() {
      common.changeTitle('订单详情');
      this.getOrderInfo();
      this.from = this.$route.query.from;
    },
    watch: {
      msg(curVal) {
        if(/[^\d]/g.test(curVal)) {
          this.msg = this.msg.replace(/[^\d]/g, '');
        }
        if(this.msg.length == 6){
          // this.submitOrder();
          this.payOrder();
        }
      },
    },
    computed:{
      totalProductNum: function(){
        let num_product = 0;
        this.order_info.order_part.forEach(item => {
          // console.log(item.prtitle);
          num_product=num_product+item.opnum;
        });
        // for(let item of this.order_info.order_part){
        //   console.log(item.prtitle);
        //   num_product=num_product+item.opnum;
        // }
        return num_product;
      }
    },
    methods: {
      changeRoute(v, item) {
        if(this.from !== 'activityProduct') {
          switch (v) {
            case '/brandDetail':
              this.$router.push({ path: v, query: { pbid: this.order_info.pbid, pbname: this.order_info.pbname }});
              break;
            case '/productDetail':
              if(this.order_info.omlogistictype == 10) {
                this.$router.push({ path: '/gift', query: { prid: item.prid }});
              }else if (this.order_info.omfrom == 80){
                this.$router.push({path:'/personal/starProductDetail',query:{ipid:item.prid}});
              }else {
                this.$router.push({ path: v, query: { prid: item.prid }});
              }
              break;
            case '/selectBack':
              if(item != 'order') {
                let arr = [] ;
                arr.push(item);
                this.$router.push({ path: v, query: { product: JSON.stringify(arr) }});
              }else {
                this.$router.push({ path: v, query: { product: JSON.stringify(this.order_info), allOrder: 1 }});
              }
              break;
            case '/logisticsInformation':
              this.$router.push({ path: v, query: { omid: this.order_info.omid }});
              break;
            case '/backDetail':
              this.$router.push({ path: v, query: { omid: this.order_info.omid }});
              break;
            case '/addComment':
              localStorage.setItem('productComment', JSON.stringify(this.order_info));
              this.$router.push(v);
              // this.$router.push({ path: v, query: { product: JSON.stringify(this.order_info) }});
              break;
            case '/personal/editInput':
              this.$router.push({path:v,query:{from:'password',way:item}});
              break;
            default:
              this.$router.push(v);
          }
        }else {
          switch (v) {
            case '/brandDetail':
              this.$router.push({ path: v, query: { pbid: this.order_info.pbid, pbname: this.order_info.pbname }});
              break;
            case '/productDetail':
              break;
              switch(Number(localStorage.getItem('activityOrderNo'))) {
                case 0:
                  this.$router.push({ path: '/activityProductDetail', query: { fmfpid: item.fmfpid, which: 'new' }});
                  break;
                case 1:
                  this.$router.push({ path: '/productDetail', query: { prid: item.prid }});
                  break;
                case 2:
                  this.$router.push({ path: '/productDetail', query: { prid: item.prid }});
                  break;
                case 3:
                  this.$router.push({ path: '/activityProductDetail', query: { tcid: item.tcid, which: 'try' }});
                  break;
              }
              break;
            case '/logisticsInformation':
              this.$router.push({ path: v, query: { omid: this.order_info.omid }});
              break;
            case '/addComment':
              localStorage.setItem('productComment', JSON.stringify(this.order_info));
              this.$router.push(v);
              break;
            case '/personal/editInput':
              this.$router.push({path:v,query:{from:'password',way:item}});
              break;
          }
        }
      },
      //获取订单详情
      getOrderInfo() {
        axios.get(api.order_get,{
          params:{
            token: localStorage.getItem('token'),
            omid: this.$route.query.omid
          }
        }).then(res => {
          if(res.data.status == 200) {
            this.order_info = res.data.data;
            console.log(this.order_info,res.data.data)
            if(res.data.data.omstatus >= 20){
              this.getLogistic();
            }
            // 售后信息
            if(res.data.data.order_refund_apply) {
              this.refund = res.data.data.order_refund_apply;
            }
            // 判断订单中是否有商品在售后状态中
            for(let i = 0; i < res.data.data.order_part.length; i ++) {
              if(res.data.data.order_part[i].order_refund_apply) {
                this.part_refund = true;
              }
            }
            if(res.data.data.order_refund_notes) {
              this.refund_notes = res.data.data.order_refund_notes;
            }
            if(this.order_info.duration) {
              this.timeOut();       // 倒计时
            }
          }
        })
      },
      // 倒计时
      timeOut() {
        if(this.order_info.duration.substr(0, 1) > -1) {
          this.order_info.min = this.order_info.duration.substr(2, 2);
          this.order_info.sec = this.order_info.duration.substr(5, 2);
          let TIME_OUT = Number(this.order_info.min) * 60 + Number(this.order_info.sec);
          let count = TIME_OUT;
          let time = setInterval(() => {
            if(count > 0 && count <= TIME_OUT) {
              count --;
              this.order_info.sec --;
              if(this.order_info.sec < 10 && this.order_info.sec > -1) {
                this.order_info.sec = '0' + this.order_info.sec;
              }
              if(this.order_info.sec == -1) {
                this.order_info.sec = 59;
                if(this.order_info.min > 0) {
                  this.order_info.min -= 1;
                }
                if(this.order_info.min < 10) {
                  if(this.order_info.min !== '00') {
                    this.order_info.min = '0' + this.order_info.min;
                  }else {
                    this.order_info.duration = null;
                  }
                }
              }
              // 刷新视图
              this.order_info = Object.assign({}, this.order_info, { min: this.order_info.min, sec: this.order_info.sec })
            }else {
              this.order_info.duration = null;
              this.getOrderInfo();
              clearInterval(time);
            }
          }, 1000);
        }else {
          this.order_info.duration = null
        }
      },
      // 获取物流信息
      getLogistic() {
        axios.get(api.get_logistic,{
          params:{
            omid:this.$route.query.omid
          }
        }).then(res => {
          if(res.data.status == 200){
            this.logistic_info = res.data.data;
          }
        })
      },
      // 取消订单
      cancelOrder() {
        MessageBox.confirm('是否取消该订单？').then(() => {
          axios.post(api.cancle_order + '?token='+ localStorage.getItem('token'), { omid:this.$route.query.omid }).then(res => {
            if(res.data.status == 200){
              this.reload();
            }
          })
        }).catch(() => {

        });
      },
      // 确认收货
      orderConfirm() {
        MessageBox.confirm('是否确认该订单的收货？').then(() => {
          axios.post(api.order_confirm + '?token='+ localStorage.getItem('token'), { omid: this.$route.query.omid }).then(res => {
            if(res.data.status == 200){
              this.reload();
            }
          });
        }).catch(() => {

        });
      },
      // 删除订单
      deleteOrder() {
        MessageBox.confirm('是否删除该订单？').then(() => {
          axios.post(api.order_delete + '?token='+ localStorage.getItem('token'), { omid: this.$route.query.omid }).then(res => {
            if(res.data.status == 200){
              this.reload();
            }
          });
        }).catch(() => {

        });
      },
      // 请求微信支付参数
      // 请求微信支付参数
      payBtn() {
        let items = this.order_info;
        console.log(items)
        let params = { omid: items.omid, omclient: '0', opaytype: '0' };
        if(items.omfrom == 80){
          this.omtruemount = items.omtruemount;
          this.omid = items.omid;
          this.usintegral = items.usintegral;
          this.show_modal = true;
        }else{
          axios.post(api.order_pay + '?token='+ localStorage.getItem('token'), params).then(res => {
            if(res.data.status == 200) {

              this.wxPay(res.data.data.args, items.omid);

            }
          });
        }

      },
      payOrder(){
        this.$http.post(this.$api.order_pay + '?token=' +localStorage.getItem('token'),{
          omid:this.omid,
          omclient:0,
          opaytype:30,
          uspaycode:this.msg,
          omtruemount: this.omtruemount
        }).then(res => {
          Toast(res.data.message);
          this.msg = '';
          if(res.data.status == 200){
            // this.$router.push("/orderList");
            this.getOrderInfo();
            this.show_modal = false;
          }else if(res.data.message == '请输入正确的支付密码'){
            this.show_modal = true;
          }
        })
      },
      focus() {
        this.$refs.pwd.focus();
      },
      // 调起微信支付
      wxPay(data) {
        let that = this;
        function onBridgeReady() {      // 微信支付接口
          WeixinJSBridge.invoke(
            'getBrandWCPayRequest', {
              "appId": data.appId,                 // 公众号名称，由商户传入
              "timeStamp": data.timeStamp,         // 时间戳，自1970年以来的秒数
              "nonceStr": data.nonceStr,           // 随机串
              "package": data.package,             // 统一下单接口返回的prepay_id参数值
              "signType": data.signType,           // 微信签名方式
              "paySign": data.sign                 // 微信签名
            },
            function(res){
              if(res.err_msg == "get_brand_wcpay_request:ok" ){             // 支付成功
                that.getOrderInfo();
              }else if(res.err_msg == "get_brand_wcpay_request:cancel" ){   // 支付过程中用户取消
                Toast('支付已取消');
              }else if(res.err_msg == "get_brand_wcpay_request:fail" ){     // 支付失败
                Toast('支付失败');
              }
            });
        }
        if (typeof WeixinJSBridge == "undefined"){
          if( document.addEventListener ){
            document.addEventListener('WeixinJSBridgeReady', onBridgeReady, false);
          }else if (document.attachEvent){
            document.attachEvent('WeixinJSBridgeReady', onBridgeReady);
            document.attachEvent('onWeixinJSBridgeReady', onBridgeReady);
          }
        }else{
          onBridgeReady();
        }
      }
    }
  }
</script>

<style lang="less" rel="stylesheet/less" scoped>
  @import "../../../common/css/orderDetail";

</style>
