

<template>
  <div>
        
        <div style="text-align: center;">

          <el-button type="primary" @click="fetchTodayData">仅查看：今天（写）</el-button>
          <el-button type="success" @click="updateDelayTo">批量设置复习日-> </el-button>
          <el-date-picker
            v-model="delayToDatetime"
            type="datetime"
            placeholder="选择要推迟到的时间"
            :default-time="defaultTime"
          />
        </div>
        <div style="text-align: center; margin-top: 10px;">

          【语文】总数：{{total}} &nbsp; &nbsp;  
          
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
            <el-table-column width="80" prop="id" sortable label="编号"></el-table-column>
            <el-table-column width="180" prop="object_content" label="对象内容">
              <template #default="scope">
                <a href="#" 
                style="font-size: 18px;"
                @click.prevent="handleLinkClick(scope.row)">{{ scope.row.object_content }}</a>
              </template>
            </el-table-column>
            <el-table-column width="150" prop="object_reminder" label="提示"></el-table-column>
            <el-table-column width="170" prop="next_review_time" sortable label="下次时间"></el-table-column>
            <el-table-column width="80" prop="review_day_descp" sortable label="下次"></el-table-column>
            <el-table-column width="70" prop="review_count" sortable label="对"></el-table-column>
            <el-table-column width="70" prop="wrong_count" sortable label="错"></el-table-column>
            <el-table-column width="80" prop="textbook" sortable label="教材"></el-table-column>
            <el-table-column width="80" prop="grade" sortable label="年级"></el-table-column>
            <el-table-column width="80" prop="section" sortable label="章节"></el-table-column>
            <el-table-column width="200" label="操作">
                <template #default="scope">
                  <div class="button-container">
                    <el-button size="small" type="success" @click="addOneDay(scope.row)">推迟1天</el-button>
                    <el-button size="small" v-if="!scope.row.needReview" type="success" @click="setTodayReview(scope.row)">今天复习</el-button>
                  </div>
                </template>
              </el-table-column>
          </el-table>


  <el-drawer
    v-model="objectDetailsOpened"
    :title="current_object.id + ' : ' + current_object.object_content + '  ------  ' + current_object.object_reminder"
    direction="rtl"
    size="60%"
  >
    
    <el-table 
      :data="objectReviewListData"
      :row-class-name="tableRowClassName"
      >
      
      <el-table-column property="review_time" label="复习时间" width="120" />
      <el-table-column property="review_result" label="结果" width="100" />
      <el-table-column property="before_review_count" label="前-对" width="100" />
      <el-table-column property="after_review_count" label="后-对" width="100" />
      <el-table-column property="before_next_review_time" label="前-下次时间" width="120" />
      <el-table-column property="after_next_review_time" label="后-下次时间" width="120" />
      <el-table-column property="review_time_span" label="间隔（天）" width="160" />
      
    </el-table>
  </el-drawer>
  </div>
</template>

<script>
import axios from 'axios';
import { ref,onMounted } from 'vue';
import { ElTable, ElTableColumn , ElButton , ElMessage} from 'element-plus';

export default {
  name: 'Main',
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

    const current_object = ref({});
    const objectDetailsOpened = ref(false);
    const objectReviewListData = ref([]);


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
        
        const response = await axios.get('http://0.0.0.0:30088/get_today_object_list', {
          
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
        
        const response = await axios.get('http://0.0.0.0:30088/get_all_object_list_cn_w', {
          
        }, {
          headers: {
            'accept': 'application/json',
            'Content-Type': 'application/json'
          }
        });
        mainData.value = response.data.data_list;
        total.value = response.data.data_list.length;

        //计算mainData.value中的review_day_descp字段，“今天”、“明天”、“后天”的数量
        

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


    const fetchReviewRecordsData = async () => {
      try {
        
        const response = await axios.get('http://0.0.0.0:30088/get_all_object_review_records_by_obj_id', {
        params: {
          obj_id: current_object.value['id']
        },
          headers: {
            'accept': 'application/json',
            'Content-Type': 'application/json'
          }
        });

        objectReviewListData.value = response.data;

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

    const handleLinkClick = (row) => {
      console.log('链接被点击了，行数据:', row);

      current_object.value = row;
      objectDetailsOpened.value = true;

      //获取objectReviewListData数据
      fetchReviewRecordsData();

    };


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
        current_object,
        objectDetailsOpened,
        objectReviewListData,
        handleLinkClick,
        handleButtonClick,
        tableRowClassName,
        addOneDay,
        setTodayReview,
        selectChange,
        updateDelayTo,
        fetchTodayData,
        fetchReviewRecordsData
        
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



