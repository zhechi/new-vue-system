<template>
  <div ref='topWrapper'>
    <div id="echart-wrapper" ref="echartWrapper" :style="{width: '100%', height: '700px'}"></div>
    <div v-show="echartData.length === 0" class="anim" style="width: 100%; height: 400px; line-height:300px;text-align:center">
      暂无数据
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
import elementResizeDetector from 'element-resize-detector'
import { max } from '../../../../../utils/max'
import moment from 'moment'
import { mapGetters } from 'vuex'
export default {
  props: {
    selectData: {
      type: Object
    },
    isUpdate: {
      type: Boolean
    }
  },
  data () {
    return {
      echartData: [],
      legendNames: [],
      xAxisNames: [],
      maxData: ''
    }
  },
  computed: {
    ...mapGetters(['initLineId'])
  },
  created () {
    let date = new Date() - 3600 * 1000 * 24 * 30
    date = moment(date).format('YYYY-MM')
    this._runtimeAnalysis({
      lineId: this.initLineId,
      type: '1',
      month: date
    })
  },
  mounted () {
    let listenResize = elementResizeDetector()
    listenResize.listenTo(this.$refs.topWrapper, (el) => {
      this.$echarts.init(document.getElementById('echart-wrapper')).resize()
    })
  },
  watch: {
    // selectData: {
    //   deep: true,
    //   handler () {
    //     if (this.selectData.value !== '' && this.selectData.turn !== '' && this.selectData.month !== '') {
    //       this._runtimeAnalysis({
    //         lineId: this.selectData.value,
    //         type: this.selectData.turn,
    //         month: moment(this.selectData.date).format('YYYY-MM')
    //       })
    //     }
    //   }
    // },
    isUpdate () {
      if (this.isUpdate) {
        if (this.selectData.value !== '' && this.selectData.turn !== '' && this.selectData.month !== '') {
          this._runtimeAnalysis({
            lineId: this.selectData.value,
            type: this.selectData.turn,
            month: moment(this.selectData.date).format('YYYY-MM')
          })
        }
        this.$emit('isUpdateTo')
      }
    }
  },
  methods: {
    _runtimeAnalysis (params) {
      this.echartData = []
      this.legendNames = []
      this.xAxisNames = []
      this.$api['schedulingAnalysis.getLineBetweenStationsRunningTimeChartDatas'](params).then(res => {
        this.echartData = res.datas
        this.legendNames = res.legendNames
        this.xAxisNames = res.xAxisNames
        if (this.echartData.length > 0) {
          let maxBefore = []
          this.recheckArr(this.echartData).forEach(item => {
            maxBefore.push(this.addNum(item))
          })
          this.maxData = max(maxBefore) + 100
          this.$refs.echartWrapper.style.display = 'block'
          setTimeout(() => {
            this.drawLine()
          }, 100)
          this.$message.success('数据已更新')
        } else {
          this.$message.warning('暂无数据')
          this.$refs.echartWrapper.style.display = 'none'
          this.maxData = ''
        }
      })
    },
    recheckArr (arr) {
      let newArr = []
      // let newArr = new Array(arr[0].length)
      // arr
      arr[0].forEach((i, index) => {
        newArr[index] = []
      })
      newArr.forEach((i, iIndex) => {
        arr.forEach((item, itemIndex) => {
          i.push(item[iIndex])
        })
      })
      // console.log(newArr)
      return newArr
    },
    addNum (arr) {
      let totleNum = 0
      arr = arr.filter(i => i !== null)
      arr.forEach(i => {
        totleNum += i
      })
      return totleNum
    },
    drawLine () {
      let chart = this.$echarts.init(document.getElementById('echart-wrapper'))
      window.addEventListener('resize', () => { chart.resize() })
      let series = []
      this.echartData.forEach((item, index) => {
        series.push({
          name: this.legendNames[index],
          data: item,
          type: 'bar',
          stack: '总量',
          barWidth: '20',
          label: {
            normal: {
              show: true,
              position: 'inside',
              fontSize: 8
            }
          },
          itemStyle: {
            normal: {
              label: {
                formatter: function (params) {
                  if (params.value) {
                    // 图表中的数据格式化
                    let num = params.value / 60
                    return `${num.toFixed(2)}`
                  } else {
                    return 0
                  }
                }
              }
            }
          }
        })
      })
      chart.setOption({
        tooltip: {
          trigger: 'item',
          axisPointer: { // 坐标轴指示器，坐标轴触发有效
            type: 'shadow' // 默认为直线，可选为：'line' | 'shadow'
          },
          // extraCssText: 'width:350px; white-space:pre-wrap'
          formatter: function (params, ticket, callback) {
            // let res = params[0].axisValue + '时' + '<br/>'
            // let index = params[0].dataIndex
            // let myseries = series
            // for (var i = 0; i < params.length; i++) {
            //   for (var j = 0; j < myseries.length; j++) {
            //     if (params[i].seriesName === myseries[j].name) {
            //       if (myseries[j].data[index] !== null) {
            //         let num = myseries[j].data[index] / 60
            //         num = num.toFixed(2)
            //         res += `<span style="width: 10px; height: 10px; border-radius: 50%;display:inline-block; background: ${params[i].color}; margin:0 5px;"></span>` + params[i].seriesName + '：' + num + 'min' + ';'
            //       } else {
            //         res += `<span style="width: 10px; height: 10px; border-radius: 50%;display:inline-block; background: ${params[i].color};margin:0 5px;"></span>` + params[i].seriesName + '：' + '无数据' + ';'
            //       }
            //     }
            //   }
            // }
            // return res
            return params.seriesName + '<br/>' + params.name + '时：' + (params.value / 60).toFixed(2) + '分钟'
          }
        },
        legend: {
          data: this.legendNames
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '5%',
          top: '12%',
          containLabel: true
        },
        xAxis: {
          type: 'value',
          max: this.maxData,
          min: 0,
          interval: Math.floor(this.maxData / 20),
          axisLabel: { formatter: '{value} s' }
        },
        yAxis: {
          type: 'category',
          data: this.xAxisNames
        },
        series
      }, true)
    }
  }
}
</script>

<style lang="scss" scoped>
[v-cloak]
{
display: none;
}
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
