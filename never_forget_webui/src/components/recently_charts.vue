<template>
  <div>
    <!-- 图表代码 -->


    <div style=" display: flex; justify-content: center;margin-top:10px">
        开始时间： <el-date-picker
                  v-model="start_datetime"
                  type="date"
                  format="YYYY-MM-DD"
                  placeholder="开始时间"
                  :default-time="default_start_datetime"
                />
                &nbsp; ~ &nbsp;
        结束时间： <el-date-picker
                  v-model="end_datetime"
                  type="date"
                  format="YYYY-MM-DD"
                  placeholder="结束时间"
                  :default-time="default_end_datetime"
                /> 
          &nbsp;&nbsp;
          <el-button type="primary" @click="initChart">查询</el-button>`
      </div>

    <div ref="chart" style="width: 98%; height: 400px;margin-top: 20px;"></div>

  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';
import * as echarts from 'echarts';

const mainData = ref([]);


const getDateString = (n) => {
      //获取当前时间的前n天或后n天的日期，输出格式：2024-06-20
      const today = new Date();
      today.setDate(today.getDate() + n);
      const year = today.getFullYear();
      const month = String(today.getMonth() + 1).padStart(2, '0');
      const day = String(today.getDate()).padStart(2, '0');
      return `${year}-${month}-${day}`;
    };


const formatDateTime = (date) => {
    const year = date.getFullYear();
    const month = String(date.getMonth() + 1).padStart(2, '0');
    const day = String(date.getDate()).padStart(2, '0');
    const hours = String(date.getHours()).padStart(2, '0');
    const minutes = String(date.getMinutes()).padStart(2, '0');
    const seconds = String(date.getSeconds()).padStart(2, '0');
    return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
  };

const default_start_datetime = new Date(getDateString(-14));
const default_end_datetime = new Date(getDateString(0) );

const start_datetime = ref(default_start_datetime);
const end_datetime = ref(default_end_datetime);

// 获取数据的异步函数
const getDataFromApi = async () => {
  console.log('getDataFromApi in ---- ');

  if (mainData.value && mainData.value.length > 0) {
    // 如果数据已经存在，直接返回
    return mainData.value;
  }
  
  const response = await axios.get('http://0.0.0.0:30088/get_review_info_for_echart', {
    params: {
      start_date: formatDateTime(start_datetime.value),
      end_date: formatDateTime(end_datetime.value)
    },
    headers: {
      'accept': 'application/json',
      'Content-Type': 'application/json'
    }
  });

  const apiData = response.data;
  console.log('apiData:', apiData);
  console.log('getDataFromApi out ---- ');

  mainData.value = apiData;

  
  return apiData;
};

// 处理接口数据，转换成 ECharts 所需的格式
const processData = (data) => {
  console.log('processData in ---- ');

  const dates = [...new Set(data.map(item => item.rtime))]; // 获取所有唯一的日期作为 X 轴
  const types = [...new Set(data.map(item => item.rtype))]; // 获取所有唯一的类型

  const series = types.map(type => {
    return {
      name: type,
      type: 'bar',
      stack: 'total',
      data: dates.map(date => {
        // 查找对应日期和类型的复习个数，找不到则为0
        const item = data.find(i => i.rtime === date && i.rtype === type);
        return item ? item.c : 0;
      }),
      label: {
        show: true
      },
      emphasis: {
        focus: 'series'
      }
    };
  });

  console.log('processData out ---- ');

  return { dates, series };
};

const initChart = async () => {
  console.log('initChart ---- ');

  // 获取数据并等待其完成
  const apiData = await getDataFromApi();

  // 处理数据
  const chartData = processData(apiData);
  
  const myChart = echarts.init(chart.value);

  const option = {
    tooltip: {
      trigger: 'axis',
      axisPointer: {
        type: 'shadow'
      }
    },
    legend: {},
    grid: {
      left: '3%',
      right: '4%',
      bottom: '3%',
      containLabel: true
    },
    xAxis: {
      type: 'category',
      data: chartData.dates  // 使用日期作为 X 轴
    },
    yAxis: {
      type: 'value'
    },
    series: chartData.series  // 使用类型作为 series
  };

  myChart.setOption(option);
};

const chart = ref(null);

onMounted(() => {
  initChart(); // 初始化图表
});
</script>

<style scoped>
/* 样式 */
</style>
