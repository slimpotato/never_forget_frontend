<template>
  <div class="chart-container">
    <el-card>
      <div ref="chart" class="chart"></div>
    </el-card>
  </div>
</template>

<script>
import * as echarts from 'echarts';
import { onMounted, ref } from 'vue';
import axios from 'axios';

export default {
  name: 'DateChart',
  setup() {
    const chart = ref(null);
    const chartInstance = ref(null);

    const fetchDataAndRenderChart = async () => {
      try {
        const response = await axios.get('http://0.0.0.0:30088/get_valid_points_by_date');
        const { data_list } = response.data;
        if (Array.isArray(data_list)) {
          const formattedData = data_list.map(item => [item.dt, item.valid_point]);
          const minValidPoint = Math.min(...data_list.map(item => item.valid_point));
          const maxValidPoint = Math.max(...data_list.map(item => item.valid_point));
          renderChart(formattedData, minValidPoint, maxValidPoint);
        } else {
          console.error('Invalid data format', data_list);
        }
      } catch (error) {
        console.error('Failed to fetch data:', error);
      }
    };

    const renderChart = (data, min, max) => {
      if (!chartInstance.value) {
        chartInstance.value = echarts.init(chart.value);
      }

      const option = {
        title: {
          top: 30,
          left: 'center',
          text: '每日有效成长点数报表',
        },
        tooltip: {
          formatter: function (params) {
            const date = params.data[0];
            const value = params.data[1];
            return `${date} ${value}`;
          },
        },
        visualMap: {
          min: min,
          max: max,
          type: 'piecewise',
          orient: 'horizontal',
          left: 'center',
          top: 65,
          inRange: {
            color: ['#d4f1d4', '#76c76b', '#2e7d32'],
          }
        },
        calendar: {
          top: 120,
          left: 30,
          right: 30,
          cellSize: ['auto', 13],
          range: new Date().getFullYear().toString(),
          itemStyle: {
            borderWidth: 0.5,
          },
          yearLabel: { show: false },
          dayLabel: {
            firstDay: 1,
            nameMap: ['日', '一', '二', '三', '四', '五', '六'],
          },
          monthLabel: {
            nameMap: [
              '一月', '二月', '三月', '四月', '五月', '六月',
              '七月', '八月', '九月', '十月', '十一月', '十二月'
            ],
          },
        },
        series: {
          type: 'heatmap',
          coordinateSystem: 'calendar',
          data: data,
        },
      };

      chartInstance.value.setOption(option);
    };

    onMounted(() => {
      fetchDataAndRenderChart();
    });

    return {
      chart,
    };
  },
};
</script>

<style scoped>
.chart-container {
  width: 100%;
  height: 100%;
}

.chart {
  width: 100%;
  height: 500px;
}

.chart-title {
  font-size: 18px;
  font-weight: bold;
  margin-bottom: 10px;
  text-align: center;
}
</style>
