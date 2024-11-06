<template>
    <div >

        
        

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
        <el-button type="info" @click="fetchMainData('CN')" :disabled="current_subject=='CN'" >查询(语文)</el-button> &nbsp; &nbsp; 
        <el-button type="info" @click="fetchMainData('ENW')" :disabled="current_subject=='ENW'">查询(英语-写)</el-button> &nbsp;&nbsp; 
        <el-button type="info" @click="fetchMainData('ENK')" :disabled="current_subject=='ENK'">查询(英语-识)</el-button> &nbsp;&nbsp; 

        记录总数：{{total}} 
      </div>


      
      <div style=" display: flex; justify-content: center;margin-top:10px; margin-bottom: 10px;">

      <el-button v-if="current_subject=='CN'" size="large" type="primary" @click="get_today_review_files" >生成今天的复习文件(语文)</el-button>  
      <el-button v-if="current_subject=='ENW'" type="warning" @click="get_today_review_files_english_w" >生成今天的复习文件(英语-写)</el-button> 
      <el-button v-if="current_subject=='ENK'" type="warning" @click="get_today_review_files_english_k" >生成今天的复习文件(英语-识)</el-button> 
      </div>

      <div style="display: flex; justify-content: center; width: 100%;">
        <div style="width:95%">
        <el-table 
              :data="mainData"  
              >
              <el-table-column width="170" prop="record_id"  label="编号"></el-table-column>
              <el-table-column width="60" prop="subject" label="科目"></el-table-column>
              <el-table-column width="100" label="打印">
                <template #default="scope">
                  <div>
                      <el-link :href="DOWNLOAD_BASE_URL+scope.row.today_review_form_for_print_url" 
                      target="_blank">听默单</el-link>
                  </div>
                </template>
              </el-table-column>
              <el-table-column width="100" label="导入">
                <template #default="scope">
                  <div>
                      <el-link :href="DOWNLOAD_BASE_URL+scope.row.today_review_result_for_import_url" 
                      target="_blank">答案</el-link>
                  </div>
                </template>
              </el-table-column>
              <el-table-column width="180" prop="create_time"  label="创建时间"></el-table-column>
              <el-table-column width="80"  label="数量">
                <template #default="scope">
                  <div>
                    {{scope.row.correct_count}} / {{scope.row.obj_count}}
                  </div>
                </template>
              </el-table-column>
              <el-table-column width="80" prop="correct_rate"  label="正确率">
                <template #default="scope">
                  <div>
                    {{scope.row.correct_rate}}%
                  </div>
                </template>
              </el-table-column>
              <el-table-column width="80" label="操作">
                <template #default="scope">
                  <div>
                      <el-button v-if="scope.row.review_result_data==null" type="primary" @click="inputReviewResult(scope.row)">录入</el-button>
                      <el-button v-if="scope.row.review_result_data!=null" type="success" @click="ShowResult(scope.row)">结果</el-button>
                  </div>
                </template>
              </el-table-column>
        </el-table>
      </div>
    </div>

    <el-drawer
    v-model="detailsOpened"
    title="本次复习的内容"
    direction="rtl"
    size="30%"
  >
    <el-table :data="recordData">
      <el-table-column property="index" label="编号" width="60" />
      <el-table-column property="content" label="词语" width="150" />
      <el-table-column property="reminder" label="提示" width="150" />
    </el-table>
  </el-drawer>

  <el-drawer
    v-model="reviewResultOpened"
    :title="'编号:' + current_record_id"
    direction="rtl"
    size="35%"
  >
    
    
    <el-date-picker
            v-if="!isResultShow"
            v-model="currentReviewTime"
            type="datetime"
            placeholder="本次复习时间"
            format="YYYY-MM-DD HH:mm:ss"
            :default-time="defaultTime"
          />

    &nbsp; &nbsp;
    <el-button v-if="!isResultShow" type="primary" @click="submitReviewResult">提交</el-button>

    <el-table 
      :data="recordData"
      @selection-change="selectChangeReviewResult"
      :row-class-name="tableRowClassName"
      >
      <el-table-column v-if="!isResultShow"  type="selection" width="40" />
      <el-table-column property="index" label="index" width="60" />
      <el-table-column property="content" label="词语" width="150" />
      <el-table-column property="reminder" label="提示" width="150" />
      <el-table-column v-if="isResultShow" label="结果" width="60">
        <template #default="scope">
                  <div>
                    <el-image v-if="scope.row.review_result=='1'" 
                    style="width: 30px; height: 30px" src="/correct.png"></el-image>
                    <el-image v-if="scope.row.review_result=='-1'" 
                    style="width: 30px; height: 30px" src="/wrong.png"></el-image>
                  </div>
                </template>
      </el-table-column>
    </el-table>
  </el-drawer>


    </div>


  </template>
  
  <script setup>

  import { onMounted, ref } from 'vue';
  import axios from 'axios';
  import { ElButton, ElLink,ElMessage } from 'element-plus';

  const current_subject = ref('CN');
  const current_record_id = ref('');
  const current_review_result_str = ref('');
  const selectReviewResultIDs = ref([]);

  const reviewResultOpened = ref(false);
  const detailsOpened = ref(false);

  const currentReviewTime = ref(new Date());
  const defaultTime = ref(new Date());

  const isResultShow = ref(false);


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

  const DOWNLOAD_BASE_URL = 'http://0.0.0.0:30088/download_output_file?file_name=';


  const today_review_form_for_print_url = ref('');
  const today_review_result_for_import_url = ref('');

  const mainData = ref([]);
  const total = ref(0);
  const recordData = ref([]);

  const default_start_datetime = new Date(getDateString(-7) + ' 00:00:00');
  const default_end_datetime = new Date(getDateString(1) + ' 23:59:59');

  const start_datetime = ref(default_start_datetime);
  const end_datetime = ref(default_end_datetime);

  const tableRowClassName = ({ row, rowIndex }) => {
    if (row.review_result == -1 ) {
        return 'warning-row';
    }
    
    return '';
  };

  const submitReviewResult = async () => {

    if (!currentReviewTime.value) {
      ElMessage({
        showClose: true,
        message: '请选择复习时间！',
        type: 'error',
        duration: 10000
      });
      return;
    }

    if (selectReviewResultIDs.value.length === 0) {
      if (!confirm('没有选择任何的复习对象，确定全对吗？')) {
        console.log('取消提交');
        return;
      }
    }

    let review_type = '';
    if (current_record_id.value.startsWith('CN') || 
        current_record_id.value.startsWith('ENK')) {
      review_type = 'k';
    } else if (current_record_id.value.startsWith('ENW')) {
      review_type = 'w';
    }

    const data = {
            incorrect_index_list: selectReviewResultIDs.value,  
            record_id: current_record_id.value,  
            review_time: formatDateTime(currentReviewTime.value),
            review_type: review_type
        };
    
    console.log('data:', data);

    // 发送POST请求
    const response = await axios.post('http://0.0.0.0:30088/input_review_results', data, {
        headers: {
            'accept': 'application/json',
            'Content-Type': 'application/json'
        }
    });

        //提示成功
    ElMessage({
      showClose: true,
      message: '更新复习成功！' + response.data.success,
      type: 'success',
      duration: 10000
    });

    fetchMainData(current_subject.value);

    reviewResultOpened.value = false;

  };

  const selectChangeReviewResult = (selection) => {
    console.log('selection:', selection);
    selectReviewResultIDs.value = selection.map(item => item.index);
    console.log('selectReviewResultIDs:', selectReviewResultIDs.value);
  };

  const inputReviewResult = (row) => {
    isResultShow.value = false;
    reviewResultOpened.value = true;
    current_record_id.value = row.record_id;
    let correctedJsonString = row.form_data.replace(/'/g, '"');
    recordData.value = JSON.parse(correctedJsonString);
  };

  const ShowResult = (row) => {
    isResultShow.value = true;
    reviewResultOpened.value = true;
    current_record_id.value = row.record_id;
    let correctedJsonString = row.review_result_data.replace(/'/g, '"');
    recordData.value = JSON.parse(correctedJsonString);
    isResultShow.value = true;
  };

  const fetchMainData = async (subject) => {
    try {

      current_subject.value = subject;

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
          subject: current_subject.value,
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

    fetchMainData(current_subject.value);

  };

  onMounted(() => {
    fetchMainData('CN');
    });


  const get_today_review_files_english_w = async () => {
    const response = await axios.get('http://0.0.0.0:30088/get_today_review_files_english_w', {
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

    fetchMainData(current_subject.value);

  };

  //get_today_review_files_english_connection
  const get_today_review_files_english_k = async () => {
    const response = await axios.get('http://0.0.0.0:30088/get_today_review_files_english_k', {
    }, { headers: { 'accept': 'application/json', 'Content-Type': 'application/json' } });

    const DOWNLOAD_BASE_URL = 'http://0.0.0.0:30088/download_output_file?file_name=';

    today_review_form_for_print_url.value = DOWNLOAD_BASE_URL + response.data.today_review_form_for_print_url;
    today_review_result_for_import_url.value = DOWNLOAD_BASE_URL + response.data.today_review_result_for_import_url;

    //提示成功
    ElMessage({
      showClose: true,
      message: '【english-connection】生成成功！',
      type: 'success',
      duration: 10000
    });

    fetchMainData(current_subject.value);

  };


  </script>
  
  <style scoped>
        .el-table .warning-row {
          --el-table-tr-bg-color: var(--el-color-warning-light-9);
      }

  </style>
  