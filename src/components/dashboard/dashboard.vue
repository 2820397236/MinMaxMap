<template lang="html">
  <div class="dashboard">
    <div class="flex-container column">
        
        <div class="item four active"  style="transform: translate(0, 0) scale(1)">
          <heat></heat>
        </div>
    </div>
  </div>
</template>

<script>
import column from 'components/column/column'
import line from 'components/line/line'
import multipleColumn from 'components/multipleColumn/multipleColumn'
import point from 'components/point/point'
import heat from 'components/heat/heat'

export default {
  data() {
    return {
      items: []
    }
  },
  mounted() {
    this._init()
  },
  methods: {
    _resize() {
      this.$root.charts.forEach((myChart) => {
        myChart.resize()
      })
    },
    _init() {
      this.items = document.querySelectorAll('.flex-container .item')
      for (let i = 0; i < this.items.length; i++) {
        this.items[i].dataset.order = i + 1;
      }
    },
    clickChart(clickIndex) {
      let activeItem = document.querySelector('.flex-container .active')
      let activeIndex = activeItem.dataset.order
      let clickItem = this.items[clickIndex - 1]
      if (activeIndex === clickIndex) {
        return
      }
      activeItem.classList.remove('active')
      clickItem.classList.add('active')
      this._setStyle(clickItem, activeItem)
    },
    _setStyle(el1, el2) {
      let transform1 = el1.style.transform
      let transform2 = el2.style.transform
      el1.style.transform = transform2
      el2.style.transform = transform1
    }
  },
  components: {
    column,
    multipleColumn,
    point,
    heat,
    'v-line': line
  }
}

</script>

<style lang="stylus" scoped>
*
  box-sizing: border-box;
.point,.multipleColumn,.columnChart,.line
  height 100%!important
  width 100%!important
  background none!important
.item
    padding: 0px;
    margin: 0px;
    width: 68%;
    height: 100%;
    position absolute
    transform scale(0.33)
    text-align: center;
    transition:all 0.8s;
    background rgba(32, 32, 35, 0.5)
.dashboard
    position relative
    width 100%
    height 100%
    margin:0px;
    padding:0px;
    padding-top 5%
    background url('../../assets/bg.jpg');
    background-size 100% 100%
.flex-container.column
    position relative
    height: 90%;
    width: 90%;
    overflow: hidden;
    margin:  0 auto 100px auto;
    box-sizing: content-box;
.active
    height 100%
    width: 100%;
    /*margin-left: 10px;*/
    line-height: 300px;
.list
    li
        height 120px;
        line-height 120px;
        font-size 40px;
        color #000;
        background-color: #fff;
.title
  position relative
  display flex
  height 50px
  line-height 50px
  background-color rgba(32, 32, 35, 0.2)
  color white
  width 100%
  h1
    flex 0 0 120px
    font-size 21px
    font-weight bold
    padding-left 20px
  ul
    position absolute
    right 0
    padding-right 20px
    margin-top -2px
    li
      display inline-block
      min-width 59px
      padding 2px 10px 2px 10px
      line-height 20px
      text-align center
      font-size 11px
      &:first-child
        border-top-left-radius 5px
        border-bottom-left-radius 5px
      &:last-child
        border-top-right-radius 5px
        border-bottom-right-radius 5px
      &+li
        margin-left: -1px
</style>
