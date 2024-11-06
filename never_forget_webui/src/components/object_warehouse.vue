<template>
    <div>
        
        <div style="margin: 10px;"> 
            当前词语分类: &nbsp; <span style="font-weight: bold;"> {{filterData.title}}</span> &nbsp;&nbsp; &nbsp;&nbsp;  
            当前科目： &nbsp; <span style="font-weight: bold;"> {{filterData.subject}}</span> &nbsp;&nbsp; &nbsp;&nbsp;
            数量: <span style="font-weight: bold;">  {{total}} </span> &nbsp;&nbsp; &nbsp;&nbsp;  
            <el-button type="primary" @click="addObjectsOpen" >添加生词</el-button>  &nbsp;&nbsp; 
            <el-button type="warning" @click="batchStart('k')" >开始学（识）</el-button>  &nbsp;&nbsp; 
            <el-button type="warning" @click="batchStart('w')" >开始学（写）</el-button>  
        </div>
          
            <el-table 
            :data="mainData"  
            @selection-change="selectChange"
            size="small"
              >
              <el-table-column width="40" type="selection"/>
              <el-table-column width="60" prop="id" sortable label="编号"></el-table-column>
              <el-table-column width="150" prop="object_content" label="对象内容"></el-table-column>
              <el-table-column width="150" prop="object_reminder" label="提示"></el-table-column>
              <el-table-column width="180" prop="next_review_time" sortable label="下次时间"></el-table-column>
              <el-table-column width="80" prop="review_day_descp" sortable label="下次"></el-table-column>
              <el-table-column width="70" prop="review_count" sortable label="对"></el-table-column>
              <el-table-column width="70" prop="wrong_count" sortable label="错"></el-table-column>
              <el-table-column width="100" prop="textbook" sortable label="教材"></el-table-column>
              <el-table-column width="100" prop="grade" sortable label="年级"></el-table-column>
              <el-table-column width="80" prop="section" sortable label="章节"></el-table-column>
              <el-table-column width="60" label="识">
                <template #default="scope">
                  <div class="button-container">
                    <span v-if="scope.row.k">已开始</span>
                    <el-button v-if="!scope.row.k" type="success" @click="oneStart(scope.row.id,'k')" size="small">开:识</el-button>
                   </div>
                </template>
              </el-table-column>
              <el-table-column width="60" label="写">
                <template #default="scope">
                  <div class="button-container">
                    <span v-if="scope.row.w">已开始</span>
                    <el-button v-if="!scope.row.w" type="success" @click="oneStart(scope.row.id,'w')" size="small">开:写</el-button>
                   </div>
                </template>
              </el-table-column>
             
            </el-table>

            <el-drawer
                v-model="addFormOpened"
                title="添加"
                direction="rtl"
                size="30%"
            >
                <el-form :model="addForm" label-width="80px">
                    <el-form-item label="科目" prop="subject">
                        <el-input v-model="addForm.subject" ></el-input>
                    </el-form-item>
                    <el-form-item label="对象内容" prop="object_content">
                        <el-input
                            v-model="addForm.object_content"
                            style="width: 280px"
                            :rows="5"
                            type="textarea"
                            placeholder="输入生词，逗号分隔"
                        />
                    </el-form-item>
                    <el-form-item label="提示" prop="object_reminder">
                        <el-input v-model="addForm.object_reminder"></el-input>
                    </el-form-item>
                    <el-form-item label="教材" prop="textbook">
                        <el-input v-model="addForm.textbook"></el-input>
                    </el-form-item>
                    <el-form-item label="年级" prop="grade">
                        <el-input v-model="addForm.grade"></el-input>
                    </el-form-item>
                    <el-form-item label="章节" prop="section">
                        <el-input v-model="addForm.section"></el-input>
                    </el-form-item>
                    <el-form-item label="识" prop="k">
                        <el-switch v-model="addForm.k" />
                    </el-form-item>
                    <el-form-item label="写" prop="w">
                        <el-switch v-model="addForm.w" />
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="submitAddForm">添加</el-button>
                        <el-button @click="resetAddForm('addForm')">重置</el-button>
                    </el-form-item>
                </el-form>
            </el-drawer>
    </div>
  </template>

<script setup>

    import { onMounted, ref } from 'vue';
    import axios from 'axios';
    import { ElButton, ElLink,ElMessage } from 'element-plus';


    const props = defineProps({
        menuIndex: {
            type: String,
            required: true,
        },
        filterData: {
            type: Object,
            required: true,
        },
    });

    const mainData = ref([]);
    const total = ref(0);
    const addFormOpened = ref(false);
    const selectIDs = ref([]);

    const addForm = ref({
        subject: '',
        object_content: '',
        object_reminder: '',
        textbook: '',
        grade: '',
        section: '',
        k: '0',
        w: '0'
    });

    const selectChange = (selection) => {
        console.log(selection);
        selectIDs.value = selection.map(item => item.id);
        console.log(selectIDs.value);
    };

    const oneStart = async (id, type) => {

        //清空选择
        selectIDs.value = [id];

        batchStart(type);
    }


    const batchStart = async (type) => {
        console.log(type);

        if (selectIDs.value.length === 0) {
            ElMessage({
                showClose: true,
                message: '请选择对象！',
                type: 'error',
                duration: 10000
            });
            return;
        }

        const data = {
            obj_id_list: selectIDs.value,  
            plan_type: type
        };
        
        // 发送POST请求
        const response = await axios.post('http://0.0.0.0:30088/start_plan_objects', data, {
            headers: {
                'accept': 'application/json',
                'Content-Type': 'application/json'
            }
        });

        ElMessage({
            showClose: true,
            message: '录入成功！' + response.data.success,
            type: 'success',
            duration: 10000
        });

        fetchMainData();

    };

    const addObjectsOpen = () => {
        addFormOpened.value = true;
    };

    const submitAddForm = async () => {
    
        if (addForm.value.subject === '' ){
            ElMessage({
                showClose: true,
                message: '科目加载失败，请刷新重试！',
                type: 'error',
                duration: 10000
            });
            return;
        }

        if (addForm.value.object_content === '' ){
            ElMessage({
                showClose: true,
                message: '请输入对象内容！',
                type: 'error',
                duration: 10000
            });
            return;
        }

        const data = {
                object_content_str: addForm.value.object_content,  
                object_reminder: addForm.value.object_reminder,
                subject: addForm.value.subject,
                textbook: addForm.value.textbook,
                grade: addForm.value.grade,
                section: addForm.value.section,
                k: addForm.value.k ? '1' : '0',
                w: addForm.value.w ? '1' : '0'
            };
        
        console.log('data:', data);

        // 发送POST请求
        const response = await axios.post('http://0.0.0.0:30088/batch_init_object_list', data, {
            headers: {
                'accept': 'application/json',
                'Content-Type': 'application/json'
            }
        });

            //提示成功
        ElMessage({
            showClose: true,
            message: '录入成功！' + response.data.result_len,
            type: 'success',
            duration: 10000
        });

        fetchMainData();

        addForm.value.object_content = '';
        addForm.value.object_reminder = '';

        //reviewResultOpened.value = false;
    };

    const fetchMainData = async () => {
      try {

        addForm.value.subject = props.filterData.subject;
        addForm.value.textbook = props.filterData.textbook;
        addForm.value.grade = props.filterData.grade;
        addForm.value.section = props.filterData.section;
        
        const response = await axios.get('http://0.0.0.0:30088/get_all_object_list_search', {
            params: {
                subject: props.filterData.subject,
                textbook: props.filterData.textbook,
                grade: props.filterData.grade,
                section: props.filterData.section,
                k: props.filterData.k,
                w: props.filterData.w
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

    onMounted(() => {

        fetchMainData();
        });

</script>