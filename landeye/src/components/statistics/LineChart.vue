<template>
  <div class='container3' >
     <div class="data_title">告警类型变化</div>
    <div ref="chart" id="chartLineBox" style="width: 100%; height: 100%;"></div>
  </div>
</template>

<script>
import * as echarts from 'echarts'
import * as Date from 'ant-design-vue';
export default {
  data() {
    return {
      startDate: '',
      endDate: '',
      // 指定图表的配置项和数据
      option: {
        legend: {
          icon: 'rect',
          itemWidth: 10,itemHeight: 5,itemGap:8,
          data: ['工程车辆进入', '人为破坏', '焚烧纵火'],
          left: 'center',bottom:'1%',
          textStyle: {fontSize: '15px',color: 'grey'}
        },
        tooltip: { // 设置tip提示
          trigger: 'item',
          show:true,
          backgroundColor: "rgba(255,255,255,0.1)",
          axisPointer: {
            type: "none"
          },
          formatter: function (params) {
            return '数量: ' + params.value;
          }
        },
        grid: {
          top: "60vh",
          left: "3%",
          right: "50vw",
          bottom: "15%",
          containLabel: true,
        },
        xAxis: { // 设置x轴
          offset:10,
          type: 'category',
          boundaryGap: false, // 坐标轴两边不留白
          data:  ['5月1日','5月2日','5月3日','5月4日','5月5日','5月6日','5月7日'],
          name: '日期', // X轴 name
          nameTextStyle: { // 坐标轴名称的文字样式
            fontSize: 16,
            padding: [0, 0, 0, -10],
            color:'grey'
          },
          axisLabel:{
            color: "grey",
          },
          axisLine:{
            show: false,
          },
          axisTick: {
            show: false,
            length: 25,
            lineStyle: {
              color: "#61B087",
            },
          },
          splitLine: {
            show: false,
            lineStyle: {
              color: "#61B087",
            },
          },
        },
        yAxis: {
          name: '数量',
          nameTextStyle: {
            fontSize: 16,
            padding: [0, 0, 0, -50],
            color:'grey'
          },
          type: 'value',
          axisLabel:{
            margin: 20,
            color: "#61B087",
          },
          axisLine:{
            lineStyle: {
              color: "#61B087",
              width: 2,
            },
          },
          axisTick: {
            show: true,
            length: 15,
            lineStyle: {
              color: "#61B087",
            },
          },
          splitLine:{
            show: true,
            lineStyle: {
              color: "#61B087",
            },
          }
        },
        series: [
          {
              name: '工程车辆进入',
              data: [40, 70, 80, 30, 60, 50, 90],
              type: 'line',
              smooth:'true',
              showAllSymbol: true,
              symbol: "circle",
              symbolSize: 6,
              lineStyle: {normal: { color: "#fff" }},
              yAxisIndex:0,
              areaStyle: {
                normal: {
                  color: new echarts.graphic.LinearGradient(
                      0,
                      0,
                      0,
                      1,
                      [
                        {
                          offset: 0,
                          color: "#61B087",
                        },
                        {
                          offset: 1,
                          color: "#3fbbff0d",
                        },
                      ],
                      false
                  ),
                },
              },
              itemStyle: {
                color: "red",
                borderColor: "#fff",
                borderWidth: 1,
              },
            },
          {
            name: '人为破坏',
            data: [70, 40, 50, 80, 30, 90, 50],
            type: 'line',
            smooth:'true',
            showAllSymbol: true,
            symbol: "circle",
            symbolSize: 6,
            lineStyle: {normal: { color: "#eff8f3" }},
            yAxisIndex:0,
            areaStyle: {
              normal: {
                color: new echarts.graphic.LinearGradient(
                    0,
                    0,
                    0,
                    1,
                    [
                      {
                        offset: 0,
                        color: "#61B087",
                      },
                      {
                        offset: 1,
                        color: "#3fbbff0d",
                      },
                    ],
                    false
                ),
              },
            },
            itemStyle: {
              color: "blue",
              borderColor: "#fff",
              borderWidth: 1,
            },
          },
          {
            name: '焚烧纵火',
            data: [10, 25, 35, 12, 55, 87, 63],
            type: 'line',
            smooth:'true',
            showAllSymbol: true,
            symbol: "circle",
            symbolSize: 6,
            lineStyle: {normal: { color: "#fff" }},
            yAxisIndex:0,
            areaStyle: {
              normal: {
                color: new echarts.graphic.LinearGradient(
                    0,
                    0,
                    0,
                    1,
                    [
                      {
                        offset: 0,
                        color: "#61B087",
                      },
                      {
                        offset: 1,
                        color: "#3fbbff0d",
                      },
                    ],
                    false
                ),
              },
            },
            itemStyle: {
              color: "green",
              borderColor: "#fff",
              borderWidth: 1,
            },
          },
          ]
      }
    }
  },
  mounted() {
    this.chartChange()
  },
  methods: {
    chartChange() {
      const myEcharts = this.$echarts.init(document.getElementById('chartLineBox'), 'vintage')
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
 .container3 {
   display: flex;
   flex-direction: column;
   position:relative;
   width: 100%;
   height: 48%;
   background: #f1f8f0;
   border-radius: 40px;
   margin-left:0;
   font-family: Arial, sans-serif;
 }
 .datechoice2 {
  padding-left: 0px
 }
 .data_title{
   width: 100%;
   height: 10%;
   text-align: center;
   color: #000000;
   font-size: 2rem;
   font-Weight: bold;
   line-height: 40px;
 }
</style>