<template>
    <div >

        
      
        <div style=" display: flex; justify-content: center;margin-top:10px; margin-bottom: 30px;">

          <el-button type="primary" @click="get_today_review_files" >生成今天的复习文件(语文)</el-button>  
          <el-button type="warning" @click="get_today_review_files_english" >生成今天的复习文件(英语)</el-button> 

          <br/>

        <el-link v-if="today_review_form_for_print_url" 
          :href="today_review_form_for_print_url" target="_blank">点击下载复习表单excel（打印需保存为PDF）</el-link> 
        &nbsp;&nbsp;&nbsp;&nbsp;
        <el-link v-if="today_review_result_for_import_url" 
          :href="today_review_result_for_import_url" target="_blank">点击下载复习结果excel（导入用）</el-link>
      
        </div>


      <div style=" display: flex; justify-content: center;margin-top:10px">
        开始时间： <el-date-picker
                  v-model="start_datetime"
                  type="datetime"
                  format="YYYY-MM-DD HH:mm:ss"
                  placeholder="选择要推迟到的时间"
                  :default-time="default_start_datetime"
                />
                &nbsp; ~ &nbsp;
        结束时间： <el-date-picker
                  v-model="end_datetime"
                  type="datetime"
                  format="YYYY-MM-DD HH:mm:ss"
                  placeholder="选择要推迟到的时间"
                  :default-time="default_end_datetime"
                /> 
                &nbsp; &nbsp;
        <el-button type="primary" @click="fetchMainData">查询</el-button> &nbsp; 记录总数：{{total}} 
      </div>

      <div style="display: flex; justify-content: center; width: 100%;">
        <div style="width:95%">
        <el-table 
              :data="mainData"  
              >
              <el-table-column width="180" prop="record_id"  label="编号"></el-table-column>
              <el-table-column width="60" prop="subject" label="科目"></el-table-column>
              <el-table-column width="60" label="数据">
                <template #default="scope">
                  <div>
                      <el-link @click="ShowDetails(scope.row)" target="_blank">查看</el-link>
                  </div>
                </template>
              </el-table-column>
              <el-table-column width="120" label="打印模板">
                <template #default="scope">
                  <div>
                      <el-link :href="DOWNLOAD_BASE_URL+scope.row.today_review_form_for_print_url" 
                      target="_blank">下载（PDF）</el-link>
                  </div>
                </template>
              </el-table-column>
              <el-table-column width="120" label="答案模板">
                <template #default="scope">
                  <div>
                      <el-link :href="DOWNLOAD_BASE_URL+scope.row.today_review_result_for_import_url" 
                      target="_blank">下载（Excel）</el-link>
                  </div>
                </template>
              </el-table-column>
              <el-table-column width="100" prop="review_result_data"  label="复习结果"></el-table-column>
              <el-table-column width="100" prop="review_result_photo_file_path"  label="拍照"></el-table-column>
              <el-table-column width="180" prop="create_time"  label="创建时间"></el-table-column>
              <el-table-column width="180" prop="update_time"  label="更新时间"></el-table-column>
        </el-table>
      </div>
    </div>


    </div>


  </template>
  
  <script setup>

  import { onMounted, ref } from 'vue';
  import axios from 'axios';
  import { ElButton, ElLink,ElMessage } from 'element-plus';


  const getDateString = (n) => {
      //获取当前时间的前n天或后n天的日期，输出格式：20240620
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

  const DOWNLOAD_BASE_URL = 'http://0.0.0.0:30088/download_output_file?file_name=';


  const today_review_form_for_print_url = ref('');
  const today_review_result_for_import_url = ref('');


  const mainData = ref([]);
  const total = ref(0);

  const default_start_datetime = new Date(getDateString(-7) + ' 00:00:00');
  const default_end_datetime = new Date(getDateString(1) + ' 23:59:59');

  const start_datetime = ref(default_start_datetime);
  const end_datetime = ref(default_end_datetime);



  const ShowDetails = (row) => {
    console.log(row);
    //TODO 像是详细信息
  };


  const fetchMainData = async () => {
    try {

      console.log('start_datetime:', start_datetime.value);
      console.log('end_datetime:', end_datetime.value);
      //将时间转换为字符串
      //格式："YYYY-MM-DD HH:mm:ss"
      const start_str = formatDateTime(new Date(start_datetime.value));
      const end_str = formatDateTime(new Date(end_datetime.value));

      console.log('formattedStartTime:', start_str);
      console.log('formattedEndTime:', end_str);

      const response = await axios.get('http://0.0.0.0:30088/get_review_form_print_records_list', {
        params: {
          subject: 'EN',
          start_time: start_str,
          end_time: end_str
        },
        headers: {
          'accept': 'application/json',
          'Content-Type': 'application/json'
        }
      });
      mainData.value = response.data.data_list;
      total.value = response.data.data_list.length;
    } catch (error) {
      console.error('请求失败:', error);
    }
  };

  const get_today_review_files = async () => {
    const response = await axios.get('http://0.0.0.0:30088/get_today_review_files', {
    }, { headers: { 'accept': 'application/json', 'Content-Type': 'application/json' } });


    today_review_form_for_print_url.value = DOWNLOAD_BASE_URL + response.data.today_review_form_for_print_url;
    today_review_result_for_import_url.value = DOWNLOAD_BASE_URL + response.data.today_review_result_for_import_url;

    //提示成功
    ElMessage({
      showClose: true,
      message: '【语文】生成成功！',
      type: 'success',
      duration: 10000
    });

  };

  onMounted(() => {
    fetchMainData();
    });


  const get_today_review_files_english = async () => {
    const response = await axios.get('http://0.0.0.0:30088/get_today_review_files_english', {
    }, { headers: { 'accept': 'application/json', 'Content-Type': 'application/json' } });

    const DOWNLOAD_BASE_URL = 'http://0.0.0.0:30088/download_output_file?file_name=';

    today_review_form_for_print_url.value = DOWNLOAD_BASE_URL + response.data.today_review_form_for_print_url;
    today_review_result_for_import_url.value = DOWNLOAD_BASE_URL + response.data.today_review_result_for_import_url;

    //提示成功
    ElMessage({
      showClose: true,
      message: '【english】生成成功！',
      type: 'success',
      duration: 10000
    });

  };




  </script>
  
  <style scoped>

  </style>
  