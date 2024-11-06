

<template>
    <div>
          
          <div style="text-align: center;">
  
            <el-button type="primary" @click="fetchTodayData">仅查看：今天（识）</el-button>
            <el-button type="success" @click="updateDelayTo">批量设置复习日-> </el-button>
            <el-date-picker
              v-model="delayToDatetime"
              type="datetime"
              placeholder="选择要推迟到的时间"
              :default-time="defaultTime"
            />
          </div>
          <div style="text-align: center; margin-top: 10px;">
  
            【english-k】总数：{{total}} &nbsp; &nbsp;  
          
          今天：<span style="color: red;"> {{todayCount}}</span> &nbsp; &nbsp;
          明天：<span style="color: red;"> {{tomorrowCount}}</span> &nbsp; &nbsp;
          后天：<span style="color: red;"> {{dayAfterTomorrowCount}}</span> &nbsp; &nbsp;

  
          </div>
            <el-table 
              :data="mainData"  
              :row-class-name="tableRowClassName"
              @selection-change="selectChange"
              size="small"
              >
              <el-table-column width="40" type="selection"/>
              <el-table-column width="60" prop="id" sortable label="编号"></el-table-column>
              <el-table-column width="150" prop="object_content" label="对象内容"></el-table-column>
              <el-table-column width="150" prop="object_reminder" label="提示"></el-table-column>
              <el-table-column width="180" prop="next_review_time" sortable label="下次时间"></el-table-column>
              <el-table-column width="80" prop="review_day_descp" sortable label="下次"></el-table-column>
              <el-table-column width="60" prop="review_count" sortable label="对"></el-table-column>
              <el-table-column width="60" prop="wrong_count" sortable label="错"></el-table-column>
              <el-table-column width="80" prop="textbook" sortable label="教材"></el-table-column>
              <el-table-column width="80" prop="grade" sortable label="年级"></el-table-column>
              <el-table-column width="65" prop="section" sortable label="章节"></el-table-column>
              <el-table-column width="220" label="操作">
                  <template #default="scope">
                    <div class="button-container">
                      <el-button size="small" type="success" @click="addOneDay(scope.row)">推迟1天</el-button>
                      <el-button size="small" v-if="!scope.row.needReview" type="success" @click="setTodayReview(scope.row)">今天复习</el-button>
                    </div>
                  </template>
                </el-table-column>
            </el-table>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  import { ref,onMounted } from 'vue';
  import { ElTable, ElTableColumn , ElButton , ElMessage} from 'element-plus';
  
  export default {
    name: 'Main_en',
    components: {
      ElTable,
      ElTableColumn,
      ElButton,
      ElMessage
    },
    setup() {
      const mainData = ref([]);
      const total = ref(0);
      const delayToDatetime = ref(null);
      const defaultTime = new Date(2000, 1, 1, 18, 18, 18)
      const selectIDs = ref([]);
  

      const todayCount = ref(0);
      const tomorrowCount = ref(0);
      const dayAfterTomorrowCount = ref(0);

  
      const formatDatetime = (date) => {
        const year = date.getFullYear();
        const month = String(date.getMonth() + 1).padStart(2, '0'); // 月份从0开始，所以需要+1
        const day = String(date.getDate()).padStart(2, '0');
        const hours = String(date.getHours()).padStart(2, '0');
        const minutes = String(date.getMinutes()).padStart(2, '0');
        const seconds = String(date.getSeconds()).padStart(2, '0');
  
        return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
      }
  
      const fetchTodayData = async () => {
        try {
          
          const response = await axios.get('http://0.0.0.0:30088/get_today_object_list_english', {
            
          }, {
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
      }
  
      const fetchMainData = async () => {
        try {
          
          const response = await axios.get('http://0.0.0.0:30088/get_all_object_list_english', {
            
          }, {
            headers: {
              'accept': 'application/json',
              'Content-Type': 'application/json'
            }
          });
          mainData.value = response.data.data_list;
          total.value = response.data.data_list.length;

          todayCount.value = 0;
          tomorrowCount.value = 0;
          dayAfterTomorrowCount.value = 0;

          mainData.value.forEach(item => {
              if ( item.needReview ) {
                todayCount.value++;
              }
              else {
                if (item.review_day_descp === '明天') {
                  tomorrowCount.value++;
                } else if (item.review_day_descp === '后天') {
                  dayAfterTomorrowCount.value++;
                }
              }
          });
    
        } catch (error) {
          console.error('请求失败:', error);
        }
      };
  
      const updateReviewTime = async (ids,nextReviewTime) => {
        try {
          
          const response = await axios.post('http://0.0.0.0:30088/update_object_next_review_time', {
            obj_id_list :ids,
            next_review_time: nextReviewTime
          }, {
            headers: {
              'accept': 'application/json',
              'Content-Type': 'application/json'
            }
          });
  
          fetchMainData();
  
        } catch (error) {
          console.error('请求失败:', error);
        }
      };
  
      const selectChange = (val) => {
        console.log('selectChange:',val);
        //获得所选在行的id
        selectIDs.value = val.map(item => item.id);
      }
  
      //推迟一天
      const addOneDay = async (row) => {
        console.log('addOneDay:',row);
  
        //创建一个id数组
        const idArray = [row.id];
  
        //加一天
        const nextReviewTime = new Date(row.next_review_time);
        nextReviewTime.setDate(nextReviewTime.getDate() + 1);
  
        //将nextReviewTime转换为字符串 2021-09-09 12:12:12
        const nextReviewTimeStr = formatDatetime(nextReviewTime);
        
        updateReviewTime(idArray,nextReviewTimeStr);
  
        ElMessage({
          showClose: true,
          message: '更新成功！',
          type: 'success',
        });
        
      }
  
      //设置为今天
      const setTodayReview = async (row) => {
        console.log('setTodayReview:',row);
  
        //创建一个id数组
        const idArray = [row.id];
  
        //设置为今天
        const today = new Date();
        const nextReviewTimeStr = formatDatetime(today);
  
        updateReviewTime(idArray,nextReviewTimeStr);
  
        ElMessage({
          showClose: true,
          message: '更新成功！',
          type: 'success',
        });
  
      }
  
      //推迟到指定的时间复习
      const updateDelayTo = async () => {
        console.log('addOneDayMuti');
  
        //先校验：需要至少选择一行，并且选择了日期
        if (selectIDs.value.length == 0) {
          ElMessage({
            showClose: true,
            message: '请至少选择一行！',
            type: 'warning',
          });
          return;
        }
  
        if (delayToDatetime.value == null) {
          ElMessage({
            showClose: true,
            message: '请选择推迟到的时间！',
            type: 'warning',
          });
          return;
        }
  
        //获取选择的id
        console.log('selectIDs:',selectIDs.value);
        const idArray = selectIDs.value;
  
        //加一天
        const nextReviewTime = new Date(delayToDatetime.value);
  
        //将nextReviewTime转换为字符串 2021-09-09 12:12:12
        const nextReviewTimeStr = formatDatetime(nextReviewTime);
  
        updateReviewTime(idArray,nextReviewTimeStr);
  
        ElMessage({
          showClose: true,
          message: '更新成功！',
          type: 'success',
        });
  
      }
  
  
      const handleButtonClick = () => {
        console.log('按钮被点击了');
      };
  
      const tableRowClassName = ({ row, rowIndex }) => {
          
          if (row.needReview) {
              return 'warning-row';
          }
          
          return '';
          
      };
  
      onMounted(() => {
          fetchMainData();
          });
  
      return {
          fetchMainData,
          mainData,
          total,
          delayToDatetime,
          defaultTime,
          todayCount,
          tomorrowCount,
          dayAfterTomorrowCount,
          handleButtonClick,
          tableRowClassName,
          addOneDay,
          setTodayReview,
          selectChange,
          updateDelayTo,
          fetchTodayData
      };
    },
  };
  </script>
  
  <style >
    
  
  
    .button-container {
      display: flex;
      gap: 5px; /* 可选：设置按钮之间的间距 */
    }
  
      .el-table .warning-row {
          --el-table-tr-bg-color: var(--el-color-warning-light-9);
      }
  
      .el-table .success-row {
          --el-table-tr-bg-color: var(--el-color-success-light-9);
      }
  
  </style>
  
  
  
  