<template>
  <div class="pic-detail-content">
    <h3 class="demonstration">报警图片<span style="margin-left:20px; font-size: 16px; font-weight: 600">共{{imgList && imgList.length || 0}}张</span></h3>
    <el-carousel v-if="imgList && imgList.length > 0" type="card" height="200px" trigger="click" :autoplay="false">
      <el-carousel-item v-for="(item, index) in imgList" :key="index">
        <img :src="item.url" width="100%" height="100%" alt="图片加载失败" />
      </el-carousel-item>
    </el-carousel>
    <img v-else src="../../../../../../assets/images/noImgData.png" width="50%" style="margin-left: 25%">
  </div>
</template>

<script type="text/ecmascript-6">
// import leftWrapper from './Components/leftWrapper';
// import rightWrapper from './Components/rightWrapper';
// import moment from 'moment';
import { Carousel, CarouselItem } from 'element-ui'
export default {
  name: 'alarmContent',
  props: {
    busDetails: {
      type: Array
    }
  },
  data () {
    return {
      imgList: [],
      loading: true
    }
  },
  components: {
    // leftWrapper,
    // rightWrapper
    'el-carousel': Carousel,
    'el-carousel-item': CarouselItem
  },
  created () {
  },
  mounted () {
  },
  watch: {
    busDetails () {
      this.imgList = this.busDetails
      this.loading = false
    }
  },
  methods: {
  }
}
</script>

<style lang="scss" scoped>
.demonstration {
  margin-top: 0px;
}
</style>
