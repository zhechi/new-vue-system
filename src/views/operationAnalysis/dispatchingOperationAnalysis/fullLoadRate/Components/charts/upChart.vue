<template>
  <div ref="topWrapper">
    <div
      ref="upChartWrapper"
      id="up-chart-wrapper"
      :style="{width: '100%', height: '330px'}"
      v-loading="loading"
      element-loading-background="rgba(255, 255, 255, 0.5)"
    >
    </div>
    <div
      ref="animationDom"
      v-show="xAxisData.length === 0"
      class="anim"
      style="width: 100%; height: 330px; line-height:300px;text-align:center">
      暂无数据
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
// import { fullRateAnalysisUp } from 'server/interface'
// import elementResizeDetector from 'element-resize-detector'
import moment from 'moment'
import { max } from '../../../../../../utils/max'
import { mapGetters } from 'vuex'
export default {
  props: {
    checkData: {
      type: Object
    },
    tabTypeData: {
      type: Array
      // default () {
      //   return ['上车人数', '下车人数', '断面客流', '满载率']
      // }
    },
    isUpdateUp: {
      type: Boolean
    }
  },
  data () {
    return {
      loading: true,
      dataSource: [],
      upPersonNum: [],
      downPersonNum: [],
      passengerFlow: [],
      fullRate: [],
      xAxisData: [],
      tabType: [],
      maxNum: '',
      maxRate: '',
      defaultData: ['上车人数', '下车人数', '断面客流', '满载率']
    }
  },
  computed: {
    ...mapGetters(['initLineId'])
  },
  created () {
    const dataNow = new Date()
    const dataBefore = moment(new Date(dataNow.getTime() - 24 * 60 * 60 * 1000)).format('YYYY-MM-DD')
    window.addEventListener('resize', () => {
      this.$refs.upChartWrapper.style.width = window.innerWidth - 220 + 'px'
    })
    this._fullRateAnalysisUp({
      lineId: this.initLineId,
      type: '1',
      dateTime: dataBefore,
      startHour: '07',
      endHour: '09'
    })
  },
  mounted () {
    this.$refs.upChartWrapper.style.width = window.innerWidth - 220 + 'px'
    // let listenResize = elementResizeDetector()
    // listenResize.listenTo(this.$refs.topWrapper, (el) => {
    //   this.$echarts.init(document.getElementById('up-chart-wrapper')).resize()
    // })
  },
  watch: {
    // checkData: {
    //   deep: true,
    //   handler () {
    //     if (this.checkData.value.length !== 0 && this.checkData.date.length !== 0 && this.checkData.startTime.length !== 0 && this.checkData.endTime.length !== 0) {
    //       this._fullRateAnalysisUp({
    //         lineId: this.checkData.value,
    //         type: '1',
    //         dateTime: moment(this.checkData.date).format('YYYY-MM-DD'),
    //         startHour: this.checkData.startTime.substring(0, 2),
    //         endHour: this.checkData.endTime.substring(0, 2)
    //       })
    //     }
    //   }
    // },
    tabTypeData (newVal) {
      // this._fullRateAnalysisUp({
      //   lineId: this.checkData.value,
      //   type: '1',
      //   dateTime: moment(this.checkData.date).format('YYYY-MM-DD'),
      //   startHour: this.checkData.startTime.substring(0, 2),
      //   endHour: this.checkData.endTime.substring(0, 2)
      // });
      this.defaultData = newVal
      this.seeType()
    },
    isUpdateUp () {
      if (this.isUpdateUp) {
        if (this.checkData.value.length !== 0 && this.checkData.date.length !== 0 && this.checkData.startTime.length !== 0 && this.checkData.endTime.length !== 0) {
          this._fullRateAnalysisUp({
            lineId: this.checkData.value,
            type: '1',
            dateTime: moment(this.checkData.date).format('YYYY-MM-DD'),
            startHour: this.checkData.startTime.substring(0, 2),
            endHour: this.checkData.endTime.substring(0, 2)
          })
        }
        this.$emit('isUpdateToUp', false)
      }
    }
  },
  methods: {
    _fullRateAnalysisUp (params) {
      this.$api['schedulingAnalysis.getUpLineStationChartDatas'](params).then(res => {
        this.dataSource = res.datas
        this.xAxisData = res.xAxisNames
        this.seeType()
        if (this.xAxisData.length > 0) {
          this.loading = false
          this.$refs.upChartWrapper.style.display = 'block'
          this.drawLine()
          this.$message.success('数据已更新')
        } else {
          // this.$refs.animationDom.addClassList('anim');
          this.$message.warning('暂无数据')
          this.$refs.upChartWrapper.style.display = 'none'
        }
      })
    },
    seeType () {
      if (this.defaultData.length === 0) {
        this.tabType = []
        // console.log(123);
        this.upPersonNum = []
        this.downPersonNum = []
        this.passengerFlow = []
        this.fullRate = []
      } else {
        this.tabType = []
        let isTypeUp = this.defaultData.some(item => item === '上车人数')
        let isTypeDown = this.defaultData.some(item => item === '下车人数')
        let isPassFlow = this.defaultData.some(item => item === '断面客流')
        let isFullRate = this.defaultData.some(item => item === '满载率')
        if (isTypeUp) {
          this.upPersonNum = this.dataSource[0]
          this.tabType.push('上车人数')
        } else {
          this.upPersonNum = []
        }
        if (isTypeDown) {
          this.downPersonNum = this.dataSource[1]
          this.tabType.push('下车人数')
        } else {
          this.downPersonNum = []
        }
        if (isPassFlow) {
          this.passengerFlow = this.dataSource[2]
          this.tabType.push('断面客流')
        } else {
          this.passengerFlow = []
        }
        if (isFullRate) {
          this.fullRate = this.dataSource[3]
          this.tabType.push('满载率')
        } else {
          this.fullRate = []
        }
      }
      if (this.dataSource.length > 0) {
        this.maxNum = max([max(this.upPersonNum), max(this.downPersonNum), max(this.passengerFlow)])
        this.maxRate = max(this.fullRate)
      }
      setTimeout(() => {
        this.drawLine()
      }, 100)
    },
    drawLine () {
      let upChart = this.$echarts.init(document.getElementById('up-chart-wrapper'))
      let timeInterval = this.xAxisData
      window.addEventListener('resize', () => {
        upChart.resize()
      })
      upChart.setOption({
        // title: {
        //   text: '实时满载率',
        //   left: 'center',
        //   textStyle: {
        //     'color': '#000'
        //   }
        // },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'shadow'
          }
          // formatter: function (params, ticket, callback) {
          //   console.log(params)
          //   let valueRate = params[3].value
          //   let valueUpPersonNum = params[0].value
          //   let valueDownPersonNum = params[1].value
          //   let passengerFlow = params[2].value
          //   return params[0].name + '<br>' + '上车人数：' + valueUpPersonNum + '<br>' + '下车人数：' + valueDownPersonNum + '<br>' + '断面客流：' + passengerFlow + '<br>' + '满载率：' + valueRate + '%'
          // }
        },
        color: ['#249cf9', '#fdb628', '#67e0e3', '#eb6f49'],
        // legend: {
        //   data: this.tabType,
        //   bottom: 10,
        //   textStyle: {
        //     color: '#000'
        //   }
        // },
        xAxis: [
          {
            type: 'category',
            data: timeInterval,
            axisPointer: {
              type: 'shadow'
            },
            splitLine: {
              show: true
            },
            axisLine: {
              show: true
            },
            axisTick: {
              show: true
            },
            axisLabel: {
              inside: false,
              interval: 2,
              showMaxLabel: true,
              textStyle: {
                color: '#000',
                fontSize: '10',
                borderRadius: '6',
                display: 'block',
                transform: 'scale(0.8)'
              }
            }
          }
        ],
        yAxis: [
          {
            type: 'value',
            // name: '水量',
            min: 0,
            max: this.maxNum + this.maxNum / 5,
            interval: Math.floor(this.maxNum / 5),
            // axisLabel: {
            //     formatter: '{value} ml'
            // },
            splitLine: {
              show: false
            },
            axisLabel: {
              inside: false,
              interval: 0,
              textStyle: {
                color: '#000',
                fontSize: '16',
                borderRadius: '6'
              }
            }
          },
          {
            type: 'value',
            min: 0,
            max: this.maxRate + 10,
            interval: 20,
            splitLine: {
              show: false
            },
            axisLabel: {
              formatter: '{value}%',
              color: '#000',
              fontSize: '16',
              borderRadius: '6'
            }
          }
        ],
        series: [
          {
            name: '上车人数',
            type: 'bar',
            barWidth: '10',
            // barGap: '-100%',
            yAxisIndex: 0,
            itemStyle: {
              emphasis: {
                barBorderRadius: 20
              },
              normal: {
                barBorderRadius: 20
              }
            },
            data: this.upPersonNum
          },
          {
            name: '下车人数',
            type: 'bar',
            barWidth: '10',
            yAxisIndex: 0,
            itemStyle: {
              emphasis: {
                barBorderRadius: 20
              },
              normal: {
                barBorderRadius: 20
              }
            },
            data: this.downPersonNum
          },
          {
            name: '断面客流',
            type: 'line',
            yAxisIndex: 0,
            barWidth: '10',
            data: this.passengerFlow
          },
          {
            name: '满载率',
            type: 'line',
            yAxisIndex: 1,
            barWidth: '10',
            data: this.fullRate
          }
        ]
      }, true)
    }
  }
}
</script>

<style lang="scss" scoped>
.anim {
  animation: zy 2.5s .15s linear forwards;
}
@keyframes zy {
  0%   { transform: rotate(15deg);}
  25%  {transform: rotate(-10deg);}
  50%  { transform: rotate(5deg);}
  75%  {transform: rotate(-5deg);}
  100% { transform: rotate(0deg);}
}
</style>
