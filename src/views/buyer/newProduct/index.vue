<template>
  <div class="m-product" @touchmove="touchMove">
    <div class="m-product-head">
      <!--轮播图-->
      <div class="m-newProduct-swipe">
        <mt-swipe :auto="3000" v-if="swipe_list">
          <mt-swipe-item v-for="item in swipe_list" :key="item.ibid">
            <img :src="item.ibpic" class="img" @click="changeRoute('/productDetail', item)">
          </mt-swipe-item>
        </mt-swipe>
      </div>
      <div class="m-selected-search left">
        <div class="m-search-input-box" @click="changeRoute('/search')">
          <span class="m-icon-search"></span>
          <span class="m-search-text">{{$route.query.kw}}</span>
        </div>
        <!--        <span class="m-icon-upload" @click="changeRoute('/circle/newEdit')"></span>-->
      </div>
    </div>
    <div class="m-product-nav">
      <img src="/static/images/product/icon-product-sort.png" class="m-icon-sort" alt="">
        <span v-for="(item,index) in nav_list" class="m-nav-one" :class="item.active?'active':''" @click="navClick(index)">
          {{item.pcname}}
        </span>
      <span class="m-product-more" @click="changeRoute('/equipment/detail')">
        更多分类
        <img src="/static/images/newpersonal/icon-more.png" class="m-icon" alt="">
      </span>
    </div>
    
    <product :list="product_list"></product>
    <bottom-line v-if="bottom_show"></bottom-line>
    <shop-icon></shop-icon>
    <!--<div class="m-modal-select" v-if="show_modal" @click="changeModal('show_modal',false)">-->
<!--    <div class="m-modal-select" v-if="show_modal">-->
<!--      <div class="m-modal-state">-->
<!--        <div class="m-state-content">-->
<!--          <template v-for="(items,index) in category_list">-->
<!--            <div class="m-one-select" v-if="items.subs">-->
<!--              <p >{{items.pcname}}</p>-->
<!--              <div class="m-sku-list">-->
<!--                <span class="m-one-sku" :class="item.active?'active':''" v-for="(item,i) in items.subs" @click.stop="categoryClick(index,i)">{{item.pcname}}</span>-->
<!--              </div>-->
<!--            </div>-->
<!--          </template>-->
<!--          &lt;!&ndash;<div class="m-one-select">&ndash;&gt;-->
<!--          &lt;!&ndash;<p>防风衣/运动外套</p>&ndash;&gt;-->
<!--          &lt;!&ndash;<div class="m-sku-list">&ndash;&gt;-->
<!--          &lt;!&ndash;<input type="text" placeholder="最低价">&ndash;&gt;-->
<!--          &lt;!&ndash;<span class="m-grey">——</span>&ndash;&gt;-->
<!--          &lt;!&ndash;<input type="text" placeholder="最低价">&ndash;&gt;-->
<!--          &lt;!&ndash;</div>&ndash;&gt;-->
<!--          &lt;!&ndash;</div>&ndash;&gt;-->
<!--        </div>-->
<!--        <div class="m-state-foot">-->
<!--          <div class="m-product-detail-btn">-->
<!--            <span @click="resetPrid">重 置</span>-->
<!--            <span class="active" @click="searchProduct">确 定</span>-->
<!--          </div>-->
<!--        </div>-->
<!--      </div>-->
<!--    </div>-->
  </div>
</template>

<script>
  import product from '../components/product';
  import navList from '../../../components/common/navlist';
  import common from '../../../common/js/common';
  import axios from 'axios';
  import api from '../../../api/api'
  import {Toast} from 'mint-ui';
  import bottomLine from '../../../components/common/bottomLine';
  import shopIcon from './compoments/shopIcon';
  var scroll = (function (className) {
    var scrollTop;
    return {
      afterOpen: function () {
        scrollTop = document.scrollingElement.scrollTop || document.body.scrollTop;
        document.body.classList.add(className);
        document.body.style.top = -scrollTop + 'px';
      },
      beforeClose: function () {
        document.body.classList.remove(className);
        document.scrollingElement.scrollTop = scrollTop;
        document.body.scrollTop = scrollTop;
      }
    };
  })('scroll');
  export default {
    data(){
      return{
        nav_list:[],
        pcid: '',
        show_modal: false,
        // brand_info:null,
        pcidList:[],
        product_list:[],
        page_info:{
          page_num:1,
          page_size:10
        },
        isScroll:true,
        total_count:0,
        total_page:0,
        bottom_show:false,
        category_list:null,
        temp: [],
        swipe_list:[]
      }
    },
    components:{
      product,
      navList,
      bottomLine,
      shopIcon
    },
    mounted(){
      common.changeTitle('商城');
      this.getCategory();
      this.getSwipe();
    },
    activated() {
      this.getCategory();
      this.getSwipe();
    },
    methods:{
      //滚动加载更多
      touchMove(e){
        let scrollTop = common.getScrollTop();
        let scrollHeight = common.getScrollHeight();
        let ClientHeight = common.getClientHeight();
        if (scrollTop + ClientHeight  >= scrollHeight -10) {
          if(this.isScroll){
            this.isScroll = false;
            if(this.page_info.page_num > this.total_page){
              this.bottom_show = true;
            }else{
              for(let i=0;i<this.nav_list.length;i++){
                if(this.nav_list[i].active){
                  this.getProduct(this.nav_list[i].pcid);
                }
              }
            }
          }
        }
      },
      // 页面跳转
      changeRoute(v){
        this.$router.push(v)
      },
      //导航切换
      navClick(index){
        let arr = [].concat(this.nav_list);
        for(let i=0;i<arr.length;i++){
          arr[i].active = false;
        }
        arr[index].active = true;
        this.nav_list = [].concat(arr);
        this.page_info.page_num = 1;
        this.bottom_show = false;
        this.getProduct(this.nav_list[index].pcid);
      },
      //显示隐藏模态框
      changeModal(v,bool) {
        this[v] = bool;
        if(bool){
          scroll.afterOpen();
        }else{
          scroll.beforeClose();
        }
      },
      //获取商品列表
      getProduct(id){
        let start = this.page_info.page_num;
        let _kw = this.$route.query.kw || '';
        axios.get(api.product_list,{
          params:{
            pcid:id,
            page_size:this.page_info.page_size,
            page_num:start,
            kw:_kw,
            token:localStorage.getItem('token')
          }
        }).then(res => {
          if(res.data.status == 200){
            if(res.data.data.length >0){
              this.page_info.page_num =   this.page_info.page_num +1;
            }else{
              this.product_list = [];
              return false;
            }
            if(start >1){
              this.product_list = this.product_list.concat(res.data.data);
            }else{
              this.product_list = res.data.data;
            }
            this.isScroll = true;
            this.total_count = res.data.total_count;
            this.total_page = res.data.total_page;
          }
        },error => {
          Toast({ message: error.data.message,duration:1000, className: 'm-toast-fail' });
        })
      },
      //获取装备信息
      getCategory(){
        axios.get(api.category_list + '?deep=1').then(res => {
          if(res.data.status == 200){
            this.nav_list = res.data.data.slice(0,4);
            for(let i in this.nav_list){
              this.nav_list[i].active = false;
            }
            this.nav_list[0].active = true;
            this.getProduct(this.nav_list[0].pcid)
          }
        });
      },
    //  获取顶部轮播图
      getSwipe(){
        this.$http.get(this.$api.list_hypermarket_banner,{
          params:{
            token:localStorage.getItem('token')
          }
        }).then(res => {
          if(res.data.status == 200){
            this.swipe_list = res.data.data;
          }
        })
      }

    }
  }
</script>

<style lang="less" rel="stylesheet/less" scoped>
  @import "../../../common/css/index";
  .m-product{
    width: 100%;
    overflow-x: hidden;

    .m-product-head{
      width: 100%;
      position: relative;
      height: 250px;
      .m-selected-search{
        position: absolute;
        top:10px;
        left: 0;
        padding: 0 33px;
        width: 680px;
        .m-search-input-box{
          .flex-row(flex-start);
          background-color: rgba(0,0,0,0.4);
          color: #fff;
          .m-icon-search{
            background: url("/static/images/product/icon-product-search.png") no-repeat;
            background-size: 100%;
          }
        }
      }
    }
    .m-product-nav{
      .flex-row(space-around);
      box-shadow:0 3px 6px rgba(0,0,0,0.16);
      .m-icon-sort{
        display: block;
        width: 40px;
        height: 40px ;
      }
      .m-nav-one{
        display: inline-block;
        padding: 20px 0;
        font-size: 24px;
        color: #666666;
        &.active{
          border-bottom: 2px solid @mainColor;
          color: @mainColor;
        }
      }
      .m-product-more{
        color: #C1C1C1;
        font-size: 24px;
        .flex-row(center);
        .m-icon{
          display: inline-block;
          width: 17px;
          height: 30px;
          margin-left: 10px;
        }
      }
    }
    .m-nav-list{
      padding: 0 75px;
      box-shadow:none;
    }
    .m-modal-select{
      position: fixed;
      top: 0;
      left: 0;
      height: 100%;
      width: 100%;
      background-color: rgba(0,0,0,0.2);
      z-index: 1001;
      transition: opacity .5s;
      .m-modal-state{
        width: 563px;
        position: absolute;
        top:0;
        right: 0;
        height: 100vh;
        background-color: #fff;
        .m-state-content{
          margin-top: 50px;
          padding: 26px 30px;
          text-align: left;
          height: 75%;
          overflow-y: auto;
          .m-one-select{
            padding: 21px 0;
            border-bottom: 1px solid #ccc;
            &:last-child{
              border-bottom: none;
            }
            input{
              display: inline-block;
              width: 200px;
              height: 40px;
              background-color: #e9e9e9;
              border-radius: 30px;
              text-align: center;
              margin: 20px 10px;
            }
            .m-one-sku{
              display: inline-block;
              padding: 6px 34px;
              background-color: #E9E9E9;
              border-radius: 10px;
              margin-right: 20px;
              margin-top: 20px;
              &.active {
                color: #ffffff;
                background-color: @mainColor;
              }
            }
          }
        }
        .m-state-foot{
          position: absolute;
          bottom: 100px;
          right: 36px;
          .m-product-detail-btn{
            display: inline-block;
            height: 62px;
            line-height: 62px;
            span{
              color: #ffffff;
              display: inline-block;
              width: 171px;
              text-align: center;
              background-color: @subColor;
              margin: 0;
              border-top-left-radius: 30px;
              border-bottom-left-radius: 30px;
              &.active{
                background-color: @mainColor;
                margin-left: -8px;
                border-radius: 0 30px 30px 0;
              }
            }
          }
        }
      }
    }
    input::-webkit-input-placeholder{
      color: #999;
      text-align: center;
    }
  }
</style>
