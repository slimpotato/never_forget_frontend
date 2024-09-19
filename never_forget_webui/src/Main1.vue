<template>
    <div>
    <el-table
      :data="mainData"
      style="width: 100%"
      :row-class-name="tableRowClassName"
    >
    <el-table-column prop="id" label="编号"></el-table-column>
    <el-table-column prop="object_name" label="对象名称"></el-table-column>
    <el-table-column prop="object_content" label="对象内容"></el-table-column>
    <el-table-column width="200px" prop="object_reminder" label="提示"></el-table-column>
    <el-table-column width="200px" prop="next_review_time" label="下次复习时间"></el-table-column>
    <el-table-column width="150px" prop="review_day_descp" label="下次复习天"></el-table-column>
    <el-table-column width="150px"  prop="review_count" label="复习次数"></el-table-column>
    <!--
      <el-table-column prop="date" label="Date" width="180" />
      <el-table-column prop="name" label="Name" width="180" />
      <el-table-column prop="address" label="Address" />
    -->
    </el-table>
   </div>
  </template>
  
  <script >
    import axios from 'axios';
    import { ref,onMounted } from 'vue';
    import { ElTable, ElTableColumn , ElButton} from 'element-plus';


    export default {
        name: 'Main',
        components: {
            ElTable,
            ElTableColumn,
            ElButton
        },
        setup() {
            const mainData = ref([]);
            const total = ref(0);

            const fetchMainData = async () => {
                try {
                    
                    const response = await axios.get('http://0.0.0.0:30088/get_all_object_list', {
                    
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
            };


            const tableRowClassName = ({ row, rowIndex }) => {

                if (rowIndex === 1) {
                    return 'warning-row';
                } else if (rowIndex === 3) {
                    return 'success-row';
                }
                return '';

            };

            onMounted(() => {
                fetchMainData();
            });


            const tableData = [
                {
                date: '2016-05-03',
                name: 'Tom',
                address: 'No. 189, Grove St, Los Angeles',
                },
                {
                date: '2016-05-02',
                name: 'Tom',
                address: 'No. 189, Grove St, Los Angeles',
                },
                {
                date: '2016-05-04',
                name: 'Tom',
                address: 'No. 189, Grove St, Los Angeles',
                },
                {
                date: '2016-05-01',
                name: 'Tom',
                address: 'No. 189, Grove St, Los Angeles',
                },
            ];

            return {
                fetchMainData,
                mainData,
                total,
                tableRowClassName,
                tableData
            };
    
    
        }
    };
  </script>
  
  <style>
  .el-table .warning-row {
    --el-table-tr-bg-color: var(--el-color-warning-light-9);
  }
  .el-table .success-row {
    --el-table-tr-bg-color: var(--el-color-success-light-9);
  }
  </style>
  