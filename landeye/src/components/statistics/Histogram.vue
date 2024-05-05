<template>
  <div class="container4">
    <div class="data_title">告警数量变化</div>
    <div ref="echartsRef" id="Histogram" style="width: 100%; height: 100%;"></div>
  </div>
</template>

<script>
import * as Date from 'ant-design-vue';
export default {
  name:'TypeNum',
  data(){
    return{
      startDate: '',
      endDate: '',
      option : {

        tooltip: { // 设置tip提示
          trigger: 'item',
          backgroundColor: "rgba(255,255,255,0.1)",
          axisPointer: {
            type: "none"
          }
        },
        grid: {
          top: '40vh',
          left: '3%',
          right: '55vw',
          bottom: '10%',
          containLabel: true
        },
        legend:{},
        xAxis: {
          offset: 10,
          name:'日期',
          type: 'category',
          data: ['5月1日','5月2日','5月3日','5月4日','5月5日','5月6日','5月7日'], // X 轴数据
          axisLabel: {
            show: true,
            textStyle: {
              color: "grey",
            },
            interval: 0,
          },
          axisLine: {
            show: true
          },
          splitLine: {
            show: false,
          },
          axisTick: {
            show: false,
          },
          nameTextStyle: { // 坐标轴名称的文字样式
            fontSize: 16,
            padding: [0, 0, 0, -5],
            color:'grey'
          }
        },
        yAxis: {
          name:'数量',
          type: 'value',
          axisLabel: {
            show: true,
            textStyle: {
              color: "#b6b5ab",
            },
          },
          axisLine: {
            show: true,
          },
          splitLine: {
            show: false,
          },
          axisTick: {
            show: false,
          },
          nameTextStyle: { // 坐标轴名称的文字样式
            fontSize: 15,
            color:'grey'
          }
        },
        series: [
          {
            data: [150, 210, 120, 70, 80, 116, 140],
            type: 'bar',
            barWidth: "60%",
            barGap: "-100%",
            itemStyle: {
              normal: {
                color: {
                  type: "linear",
                  x: 1,
                  y: 1,
                  x2: 0,
                  y2: 0,
                  colorStops: [
                    {
                      offset: 0,
                      color: "#00c6f4", // 0% 处的颜色
                    },
                    {
                      offset: 1,
                      color: "#00db97", // 100% 处的颜色
                    },
                  ],
                  globalCoord: false, // 缺省为 false
                },
                borderWidth: 0,
                barBorderRadius: [50, 50, 0, 0],
              },
            },
          },
        ]
      },
    }
  },
  mounted() {
    this.chartChange()
  },
  methods: {
    chartChange() {
      const myEcharts = this.$echarts.init(document.getElementById('Histogram'), 'vintage')
      // 使用刚指定的配置项和数据显示图表。
      myEcharts.setOption(this.option, true)
      window.addEventListener('resize', () => {
        myEcharts.resize();
      });
    },
    generateDateRange() {
      const startDate = new Date(this.startDate);
      const endDate = new Date(this.endDate);
      const dateRange = [];
      // 添加起始日期
      dateRange.push(startDate.toISOString().slice(0, 7)); // 格式化为'yyyy-MM'
      // 循环生成中间日期
      while (startDate < endDate) {
        startDate.setMonth(startDate.getMonth() + 1);
        const nextMonth = new Date(startDate);
        dateRange.push(nextMonth.toISOString().slice(0, 7)); // 格式化为'yyyy-MM'
      }
      this.option.xAxis.data = dateRange;
      this.chartChange();
    }
  }
}
</script>
<style>
.container4 {
  display: flex;
  flex-direction: column;
  position:relative;
  width: 100%;
  height: 48%;
  background: #f1f8f0;
  border-radius: 4vh;
  margin-left:0;
  margin-bottom:1vh;
}
.datechoice1 {
  padding-left:16px
}
.data_title{
  width: 100%;
  height: 2vh;
  text-align: center;
  color: #000000;
  font-size: 1.5vw;
  font-Weight: bold;
  line-height: 6vh;
}
</style>


