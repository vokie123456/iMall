<template>
    <div id="search">
      <my-header >返回</my-header>
      <div class="search-bar">
        <span class="glyphicon glyphicon-search"></span>
        <input type="text" placeholder="请输入商品名称" class="form-control search" v-model="goodsName">
        <button @click="search">搜索</button>
      </div>
      <h3 class="title">推荐商品</h3>
      <ul class="itemList">
        <li v-for="item in goodsList">
          <img v-lazy="'/static/goodsImg/1.jpg'" alt="商品图片">
          <div class="goodsItem">
              <h4>{{item.name}}</h4>
              <p>{{item.disc}}</p>
              <div class="price">
                <span class="now">￥{{item.newPrice}}</span>
                <span class="ago">￥{{item.oldPrice}}</span>
              </div>
            <img src="/static/images/cart.png" alt="加入购物车" class="putCart" @click="addCart(item.id,item.newPrice)">
          </div>
        </li>
      </ul>
      <my-footer @toDao="go" ></my-footer>
    </div>
</template>

<script>
  import myHeader from '../components/myHeader'
  import myFooter from '../components/goodsFooter'
    export default {
        name: "search",
      data(){
          return {
            goodsList:[],
            totalPrice:0,
            goodsName:''
          }
      },
      components:{
        myHeader,
        myFooter
      },
      mounted(){
          this.getGoodList();
          this.getGoodsNum();
      },
      computed:{
          cartId(){
            var cookieArr = document.cookie.split('; ')[1].slice(11);
            return cookieArr;
          }
      },
      methods:{
         go(){
           if(this.$store.state.goodsNum==0){
             this.$toasted.show('购物车里还没东西',{position:'top-center',duration:1000});
             return false;
           }
           this.$router.push('/daohang')
         },
        getGoodList(){
            this.$http.post('/api/getGoodsList').then(res=>{
              let result = res.data;
              if(result.code == '1000'){
                this.goodsList = result.allGoods;
              }
            })
        },
        addCart(id,price){
            this.$http.post('/api/add_goods',{goods_id:id,trolley_id:this.cartId}).then(res=>{
            let result = res.data;
            if(result.code=='1000'){
              this.$store.commit('updateNum',1);

              this.$store.commit('updateAmount',price);
              this.$toasted.show('加入购物车成功',{position:'top-center',duration:1000,type:'success'});
            }else if(result.code=='1001'){
              this.$toasted.show("不能重复加入购物车",{position:'top-center',duration:1000,type:'error'});
            }
          })
        },
        getGoodsNum(){
          this.$http.post('/api/getTrolleyInfo',{trolley_id:this.cartId}).then(res=>{
            this.$store.state.goodsNum = res.data.count;
            this.$store.state.amount = res.data.amount;
          })
        },
        search(){
           this.$http.post('/api/search',{name:this.goodsName,trolley_id:this.cartId}).then(res=>{
             var result = res.data;
             if(result.code=='1000'){
               this.goodsList = result.searchGoods;
             }else{
               this.$toasted.show("没有该商品!",{position:'top-center',duration:1000,type:'error'})
             }
           })
        }
      }
    }
</script>

<style scoped lang="less">
  #search{
    min-height: 100vh;
    .search-bar{
      height: 0.46rem;
      border-bottom: 2px solid #EB5648;
      padding: 0.08rem 0.4rem;
      background-color: #fff;
      font-size: 0.14rem;
      position: relative;
      span{
        position: absolute;
        top: 0.16rem;
        left: 0.48rem;
        font-size: 0.16rem;
        color: #AAAAAA
      }
      .search{
        width: 2rem;
        height: 0.3rem;
        padding-left: 0.3rem;
        display: inline-block;
        border-radius: 8px;
      }
      button{
        height: 0.3rem;
        width: 0.6rem;
        color: white;
        text-align: center;
        line-height: 0.3rem;
        background-color: #EB5648;
        border-radius: 4px;
        margin-left:0.2rem;
      }
    }
    .title{
      font-size: 0.2rem;
      margin: 0.19rem 0.2rem;
      font-weight: bold;
    }
    ul{
      padding: 0 0.1rem;
      li{
        height: 1.6rem;
        border: 1px solid #bbb;
        box-shadow: 0 0 8px #aaa;
        margin-bottom: 0.17rem;
        padding: 0.2rem ;
        background-color: #fff;
        img{
          height: 1.1rem;
          width: 1.1rem;
          float: left;
        }
        .goodsItem{
          height: 1.2rem;
          width: 2rem;
          float: right;
          position: relative;
          h4{
            font-size: 0.18rem;
            font-weight: normal;
            overflow: hidden;
            white-space: nowrap;
            text-overflow: ellipsis;
            line-height: 0.26rem;
          }
          p{
            color: #EB5648;
            font-size: 0.14rem;
            line-height: 0.4rem;
          }
          .price{
            font-size: 0.16rem;
            margin-top: 0.2rem;
            .now{
              color: #EB5648;
              font-weight: bolder;
            }
            .ago{
              font-size: 0.14rem;
              color: #aaa;
              text-decoration: line-through;
            }
          }
          .putCart{
            height: 0.5rem;
            width: 0.5rem;
            position: absolute;
            right: 0;
            bottom: 0;
          }
        }
        &:last-child{
          margin-bottom: 1rem;
        }
      }
    }

  }
</style>
