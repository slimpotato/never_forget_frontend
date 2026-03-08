<template>
    <div id="date-chart" style="width: 100%; height: 400px;"></div>
  </template>
  
  <script>
  import * as echarts from 'echarts';
  import axios from 'axios';
  
  export default {
    name: 'DateReportChart',
    mounted() {
      this.fetchDataAndRenderChart();
    },
    methods: {
      async fetchDataAndRenderChart() {
        try {
          const response = await axios.get('http://0.0.0.0:30088/get_object_review_data?obj_ids=160%2C249%2C286');
          const data = response.data.data;
          const processedData = this.processData(data);
          this.renderChart(processedData);
        } catch (error) {
          console.error('Error fetching data:', error);
        }
      },
      processData(rawData) {
        const objectMap = {};
  
        rawData.forEach(item => {
          const date = item.review_date;
          const content = item.object_content;
          const result = item.review_result;
  
          if (!objectMap[content]) {
            objectMap[content] = {};
          }
  
          if (!objectMap[content][date]) {
            objectMap[content][date] = 0;
          }
  
          objectMap[content][date] += result;
        });
  
        const allDates = Array.from(new Set(rawData.map(item => item.review_date))).sort();
  
        const series = Object.keys(objectMap).map(content => {
          let cumulative = 1; // Start cumulative at 1
          const data = allDates.map(date => {
            cumulative += objectMap[content][date] || 0;
            cumulative = Math.max(cumulative, 1); // Ensure cumulative value does not drop below 1
            return cumulative;
          });
          return {
            name: content,
            type: 'line',
            data,
            smooth: true,
            endLabel: {
              show: true,
              formatter: () => content,
            },
          };
        });
  
        return { categories: allDates, series };
      },
      renderChart({ categories, series }) {
        const chart = echarts.init(document.getElementById('date-chart'));
        const option = {
          title: {
            text: 'Object Review Trends',
            left: 'center',
          },
          tooltip: {
            trigger: 'axis',
            axisPointer: {
              type: 'line',
            },
          },
          legend: {
            top: '10%',
          },
          grid: {
            left: '3%',
            right: '4%',
            bottom: '3%',
            containLabel: true,
          },
          xAxis: {
            type: 'category',
            data: categories,
            axisLabel: {
              rotate: 45,
              formatter: value => value,
            },
          },
          yAxis: {
            type: 'value',
            name: 'Cumulative Score',
          },
          series,
        };
  
        chart.setOption(option);
      },
    },
  };
  </script>
  
  <style scoped>
  #date-chart {
    width: 100%;
    height: 400px;
  }
  </style>
  