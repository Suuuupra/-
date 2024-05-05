<template>
    <div class="Rchart-container">
      <div class="chart">
      <div class="data_title">累计告警数量</div>
      <div  ref="chart1" id="chart1" style=" top:-5%;width: 100%; height: 100%;"></div>
      </div>
      <div class="radar">
        <div class="data_title">违规车辆类型数量</div>
      <div  ref="chart2" id="chart2" style="width: 100%; height: 100%;"></div>
      </div>
    </div>
</template>

<script>
import * as echarts from 'echarts';

export default {
  mounted() {
    this.renderCharts();
  },
  methods: {
    renderCharts() {
      const chart1 = echarts.init(this.$refs.chart1);
      const chart2 = echarts.init(this.$refs.chart2);

      const option1 = {
        tooltip: {
          trigger: 'item'
        },
        legend: {
          bottom: '1%',
          left: 'center',itemWidth: 10,itemHeight: 5,itemGap:8,
          textStyle: {fontSize: '15px',color: 'grey'}
        },

        series: [
          {
            name: '累计告警次数',
            type: 'pie',
            radius: ['40%', '70%'],
            avoidLabelOverlap: true,
            itemStyle: {
              borderRadius: 10,
              borderColor: '#fff',
              borderWidth: 2
            },
            label: {
              show: false,
              position: 'center'
            },

            emphasis: {
              label: {
                show: true,
                fontSize: '25px',
                fontWeight: 'bold'
              }
            },
            labelLine: {
              show: false
            },
            data: [
              { value: 735, name: '工程车辆' },
              { value: 484, name: '人为破坏' },
              { value: 300, name: '焚烧纵火' }
            ]
          }
        ]
      };
      const option2 = {
        tooltip: {
          trigger: 'item'
        },

        radar: {
          center:['50%','50%'],
          radius:'60%',
          name: {                             // (圆外的标签)雷达图每个指示器名称的配置项。
            formatter: '{value}',
            textStyle: {
              fontSize: '15px',
              color: 'grey'
            }
          },
          nameGap:'8',
          // shape: 'circle',
          indicator: [
            { name: '推土机', max: 50 },
            { name: '挖掘机', max: 50 },
            { name: '起重机', max: 50 },
            { name: '拖拉机', max: 50 },
            { name: '铲车', max: 50 },
            { name: '搅拌车', max: 50 }
          ],
        },
        series: [
          {
            name: 'Budget vs spending',
            type: 'radar',
            avoidLabelOverlap: true,
            data: [
              {
                value: [40, 32, 28, 26, 42, 21],
                name: '违规车辆数量',
                label: {
                  normal: {
                    show:true,
                    position:'top'
                  }
                }
              }
            ]
          }
        ]
      };
      chart1.setOption(option1);
      chart1.resize();
      chart2.setOption(option2)
      chart2.resize()

      window.addEventListener('resize', () => {
        chart1.resize();
        chart2.resize();
      });
    }
  }
};
</script>

<style scoped>
.Rchart-container {
  width: 48%;
  height: 100%;
  display: flex;
  flex-direction: column;
  position: relative;
}

.chart {
  width: 100%; /* Adjust width as needed */
  height: 48%;
  background: #f1f8f0;
  border-radius: 40px;
}
.radar {
  margin-top: 2vh;
  width: 100%;
  height: 48%;
  background: #f1f8f0;
  border-radius: 40px;
}
</style>
