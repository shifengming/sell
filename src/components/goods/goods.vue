<template>
  <!--商品部分-->
  <div class="goods">
    <!--左侧菜单-->
    <div class="menu-wrapper" ref="menu">
      <ul>
        <li v-for="(item,index) in goods" class="menu-item" :class="{'current': currentIndex === index}" @click="selectMenu(index,$event)">
          <span class="text border-1px">
            <span v-show="item.type === 1 || item.type === 3 || item.type === 4" class="icon" :class="classMap[item.type]"></span>{{item.name}}
          </span>
        </li>
      </ul>
    </div>
    <!--右侧列表布局-->
    <div class="foods-wrapper" ref="foods">
      <ul>
        <li v-for="item in goods" class="food-list food-list-hook">
          <h1 class="title">{{item.name}}</h1>
          <ul>
            <li @click="selectFood(food,$event)" v-for="food in item.foods" class="food-item border-1px">
              <!--商品图-->
              <div class="icon">
                <img width="57" height="57" :src="food.icon">
              </div>
              <!--商品内容-->
              <div class="content">
                <!--商品名称-->
                <h2 class="name">{{food.name}}</h2>
                <!--商品描述-->
                <p class="desc">{{food.description}}</p>
                <!--商品月售和好评率-->
                <div class="extra">
                  <span class="count">月售{{food.sellCount}}/份</span><span>好评率{{food.rating}}%</span>
                </div>
                <!--商品的价格-->
                <div class="price">
                  <span class="now">￥{{food.price}}</span><span class="old" v-show="food.oldPrice">￥{{food.oldPrice}}</span>
                </div>
                <!--添加购物车按钮-->
                <div class="cartcontrol-wrapper">
                  <cartcontrol @add="add" :food="food"></cartcontrol>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <shopcart ref="shopcart" :selectFoods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopcart>
    <!-- 商品详情组件 -->
    <food :food="selectedFood" ref="food"></food>
  </div>

</template>
<script type="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import shopcart from '../shopcart/shopcart';
  import cartcontrol from '../cartcontrol/cartcontrol';
  import food from '../food/food'

  const ERR_OK=0;

  export default {
    name: 'goods',
    props:{
      seller:{
        type: Object
      }
    },
    data() {
      return {
        goods: [],
        listHeight:[],//每个区间的高度
        scrollY: 0,
        selectedFood:{}//点击选中商品
      }
    },
    created() {
      this._getGoods()
      this.classMap=['decrease','discount','special','invoice','guarantee'];
    },
    computed:{
      // 当页面出现滚动this.scrollY会实时变化，计算属性currentIndex()也会跟着重新计算
      currentIndex(){//左侧当前索引
        //this.listHeight是一个数组，存放着每一个li的高度
        for(let i=0;i<this.listHeight.length;i++){
          let height1 = this.listHeight[i];//第一个li
          let height2 = this.listHeight[i+1];//第二个li
          //判断实时滚动记录this.scrollY是否落在第一个li和第二个li之间 加"="是为了初始化第一个左侧current为高亮
          if(!height2 || (this.scrollY >= height1 && this.scrollY < height2)){//判断是否是最后一个li或者落到这个区间
            return i;
          }
        }
        return 0;
      },
      selectFoods() {
        let foods = [];
        this.goods.forEach(good => {
          good.foods.forEach((food) => {
            if(food.count){//如果存在count.就添加一个food
              foods.push(food);
            }
          });
        });
        return foods;
      }
    },
    methods: {
      add(target) {
        console.log(target)
        this._drop(target)
      },
      selectMenu(index, event){
        // 注意： better-scroll会将点击事件去掉, 如果滚动部分需要点击事件, 需要在参数里加上click: true
        // 原生的event事件上没有_constructed这个属性
        if(!event._constructed){ //// 如果不存在这个属性, 则认为是原生点击事件, 不执行下面的函数
          return;
        }
        let foodList = this.$refs.foods.getElementsByClassName('food-list-hook');
        let el = foodList[index];
        this.foodScroll.scrollToElement(el,300);
      },
      selectFood(food,event){
        if(!event._constructed){
          return;
        }
        this.selectedFood = food;
        this.$refs.food.show();
      },
      _drop(target){
        // 体验优化，异步执行下落动画
        this.$nextTick(() => {
          // 父组件访问子组件方法
          this.$refs.shopcart.drop(target);
        });
      },
      _getGoods() {
        this.$axios.get('/api/goods').then(res => {
          console.log(res)
          if(res.data.errno == 0) {
            this.goods = res.data.data
            this.$nextTick(() => {// DOM更新后(是异步的)
              this._initScroll();
              this._calculateHeight();//计算高度
            });
          }
        }).catch(err => {
          console.log(err)
        })
      },
      _initScroll() {
        this.menuScroll = new BScroll(this.$refs.menu,{
          click: true //取消阻止的默认行为
        });
        this.foodScroll = new BScroll(this.$refs.foods,{
          click: true,
          probeType: 3 //实时告诉我滚动的位置
        });
        this.foodScroll.on('scroll',(pos)=>{
          this.scrollY = Math.abs(Math.round(pos.y));
        })
      },
      _calculateHeight(){
        let foodList = this.$refs.foods.getElementsByClassName('food-list-hook');
        let height = 0;//第一个高度
        this.listHeight.push(height);
        for(let i = 0;i<foodList.length; i++){
          let item = foodList[i];
          height += item.clientHeight;
          this.listHeight.push(height);
        }
      }
    },
    components: {
      food,
      shopcart,
      cartcontrol
    },
    event:{
      'cart.add'(target){
        this._drop(target);
      }
    }
  };
</script>
<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl"
  .goods
    display: flex
    position: absolute
    top: 174px
    bottom: 46px
    width: 100%
    overflow: hidden
    .menu-wrapper
      flex: 0 0 80px
      width: 80px  //不写宽度，在安卓手机上会出现问题
      background: #f3f5f7
      .menu-item
        display: table //垂直居中
        height: 54px
        width: 56px
        padding: 0 12px
        line-height: 14px
        &.current
          position: relative
          margin-top: -1px //覆盖1px边框
          z-index: 10
          background: #fff
          font-weight: 700
          .text
            border-none()
        .icon
          display: inline-block
          width: 12px
          height: 12px
          vertical-align: top
          margin-right: 2px
          background-size: 16px 16px
          background-repeat:  no-repeat
          &.decrease
            bg-image('decrease_3')
          &.discount
            bg-image('discount_3')
          &.guarantee
            bg-image('guarantee_3')
          &.invoice
            bg-image('invoice_3')
          &.special
            bg-image('special_3')
        .text
          display: table-cell
          width: 56px
          vertical-align: middle
          border-1px(rgba(7,17,27,0.1))
          font-size: 12px
    .foods-wrapper
      flex: 1
      .title
        padding-left: 14px;
        height: 26px;
        line-height: 26px;
        border-left: 2px solid #d9dde1
        font-size: 12px
        color:rgb(147,153,159)
        background: #f3f5f7
      .food-item
        display:flex
        margin: 18px
        padding-bottom: 18px
        border-1px(rgba(7,17,27,0.1))
        &:last-child
          border-none()
          margin-bottom: 0
        .icon
          flex: 0 0 57px
          margin-right: 10px
        .content
          flex: 1
          .name
            margin: 2px 0 8px 0
            height: 14px
            line-height: 14px
            font-size: 14px
            color: rgb(7,17,27)
          .desc, .extra
            margin-bottom: 8px
            line-height: 10px
            font-size: 10px
            color: rgb(147,153,159)
          .desc
            line-height: 12px
            margin-bottom: 8px
          .extra
            .count
              margin-right: 12px
          .price
            font-weight: 700
            font-size: 24px
            .now
              margin-right: 8px
              font-size: 14px
              color: rgb(240,20,20)
            .old
              text-decoration: line-through
              font-size: 10px
              color: rgb(147,153,159)
          .cartcontrol-wrapper
            position: absolute
            right: 0
            bottom: 12px
</style>

