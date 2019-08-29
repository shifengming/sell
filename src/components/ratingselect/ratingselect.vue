<template>
  <!--评论的全部，满意，不满意 部分-->
  <div class="ratingselect">
    <div class="rating-type border-1px">
      <span @click="select(2, $event)" class="block positive" :class="{'active': this.selectType2 === 2}">
        {{ desc.all }}
        <span class="count">{{ ratings.length }}
        </span>
      </span>
      <span @click="select(0, $event)" class="block positive" :class="{'active': this.selectType2 === 0}">
        {{ desc.positive }}
        <span class="count">{{ positives.length }}</span>
      </span>
      <span @click="select(1, $event)" class="block negative" :class="{'active': this.selectType2 === 1}">
        {{ desc.negative }}
        <span class="count">{{ negatives.length }}</span>
      </span>
    </div>
    <!-- 勾选图标 -->
    <div @click="toggleContent" class="switch" :class="{'on': this.onlyContent2}">
      <i class="icon-check_circle"></i>
      <span class="text">只看有内容的评价</span>
    </div>
  </div>
</template>
<script>
  const POSITIVE = 0; // 正向
  const NEGATIVE = 1; // 负面
  const ALL = 2; // 全部
  export default{
    props: {
      ratings: {
        type: Array, // 注意：数组的对象都要返回一个函数
        default() {
          return [];
        }
      },
      selectType: { // 选择类型
        type: Number,
        default: ALL
      },
      onlyContent: { // 只显示内容
        type: Boolean,
        default: false
      },
      desc: {
        type: Object,
        default() {
          return {
            all: '全部',
            positive: '满意',
            negative: '不满意'
          };
        }
      }
    },
    data() {
      return {
        selectType2: this.selectType,
        onlyContent2: this.onlyContent
      }
    },
    methods: {
      select(type, event) {
        if (!event._constructed) {
          return;
        }
        this.selectType2 = type;
        this.$emit('select', type);
      },
      toggleContent(event) {
        if (!event._constructed) {
          return;
        }
        this.onlyContent2 = !this.onlyContent2;
        this.$emit('toggle', this.onlyContent2);
      }
    },
    computed: {
      positives() {
        return this.ratings.filter((rating) => {
          // console.log(rating.rateType === POSITIVE); // 返回true或false
          return rating.rateType === POSITIVE
        });
      },
      negatives() {
        return this.ratings.filter((rating) => {
          return rating.rateType === NEGATIVE
        });
      }
    }
  }
</script>
<style rel="stylesheet/stylus" lang="stylus" >
  @import "../../common/stylus/mixin.styl";
  .ratingselect
    .rating-type
      padding: 18px 0
      margin: 0 18px
      border-1px(rgba(7, 17, 27, 0.1))
      font-size: 0 // 内联元素受空白区域的影响
      .block
        display: inline-block
        padding: 8px 12px
        margin-right: 8px
        border-radius: 1px
        line-height: 16px
        color: rgb(77, 85, 93)
        font-size: 12px
        &.active
          color: #fff
        &.positive
          background: rgba(0, 160, 220, .2)
          &.active
            background: rgb(0, 160, 220)
        &.negative
          background: rgba(77, 85, 93, .2)
          &.active
            background: rgb(77, 85, 93)
        .count
          font-size: 8px
          margin-left: 2px
    .switch
      padding: 12px 18px
      line-height: 24px
      border-bottom: 1px solid rgba(7, 17, 27, .1)
      color: rgb(147, 153, 159)
      font-size: 0
      &.on
        .icon-check_circle
          color: #00c850
      .icon-check_circle
        display: inline-block
        vertical-align: top
        margin-right: 4px
        font-size: 24px
      .text
        display: inline-block
        vertical-align: top
        font-size: 12px
</style>
