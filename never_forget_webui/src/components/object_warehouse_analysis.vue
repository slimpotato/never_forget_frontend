<template>
    <div>

        <table >
            <thead>
                <tr>
                    <th style="width:100px;text-align: center;">章节</th>
                    <th style="width:150px;text-align: center;">个数：均错次</th>
                    <th style="text-align: center;">单词</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="index in totalSectionList" :key="index" > 

                    <td style="text-align: center;"> 
                       <span style="font-size: 24px;font-weight: bold;  "> 第 {{ index }} 课 </span>
                    </td>
                    <td style="text-align: center;"> 
                       <span style="font-size: 16px;color: red; "> {{ getObjectCountBySection(index) }} 个： {{ getWrongRateBySection(index) }} 次 </span>
                    </td>
                    <td> 
                        <div style="margin-top: 10px;margin-bottom: 10px;">
                            <span v-for="obj in getObjectsBySection(index)" :key="obj.id"  class="rounded-border" :style="getSpanStyle(obj)">
                            <el-tooltip :content="getTooltipContent(obj)" placement="top" effect="light">
                                <span >
                                    {{ obj.object_content }}
                                </span> 
                            </el-tooltip>
                            </span>
                        </div>
                    </td>

                </tr>
            </tbody>
        </table>

    </div>
</template>
  
<script setup>
    import { ref, onMounted } from 'vue';
    import axios from 'axios';

    const totalSectionList = ref([]);

    const mainData = ref([]);

    const getBGColorByWrongCount= (value) => {
        const colors = [
            '#FFFFFF',
            '#FFFFE3',
            '#FFFFC7',
            '#FFFFAA',
            '#FFFF8E',
            '#FFFF71',
            '#FFFF55',
            '#FFFF38',
            '#FFFF1C',
            '#FFFF00'
        ];

        if (value == null || value <= 1) {
            return colors[0]; // 当 value 为 null、0 或 1 时，返回黄色
        }

        return colors[Math.min(value - 1, colors.length - 1)]; // 返回对应的颜色，最多为绿色
    };

    const getTooltipContent = (obj) => {
        return `${obj.object_reminder} | 
         ${obj.review_day_descp} | 
         错 ${obj.wrong_count} 次 `;
    };

    const getObjectCountBySection = (section) => {
        return mainData.value.filter(item => item.section === section).length;
    };

    const getObjectsBySection = (section) => {
        let data = mainData.value.filter(item => item.section === section);

        //按照wrong_count排序
        data.sort((a, b) => b.wrong_count - a.wrong_count);
        return data;
    };


    const getWrongRateBySection = (section) => {
        let data = mainData.value.filter(item => item.section === section);

        let total = data.length;
        let totalWrongCount = data.reduce((acc, current) => {
            return acc + current.wrong_count;
        }, 0);

        let wrongRate = (totalWrongCount / total ).toFixed(1);

        return wrongRate;
    };

    const getSpanStyle = (obj) => {

        return {
            backgroundColor: getBGColorByWrongCount(obj.wrong_count),
            color:  'black'
        };
    };


    const fetchMainData = async () => {
        try {

            mainData.value = [];
            
            const response = await axios.get('http://0.0.0.0:30088/get_all_object_list_search', {
                params: {
                    subject: '语文',
                    textbook: '人教版',
                    grade: '五年级上',
                    k: 1,
                },
                headers: {
                    'accept': 'application/json',
                    'Content-Type': 'application/json'
                }
            });

            mainData.value = response.data.data_list;

            //获取totalSectionList，从mainData中获取section，去重
            totalSectionList.value = mainData.value.reduce((acc, current) => {
                const x = acc.find(item => item === current.section);
                if (!x) {
                    return acc.concat([current.section]);
                } else {
                    return acc;
                }
            }, []);

            //排序,将section转换为数值类型，按照从小到大排序
            totalSectionList.value.sort((a, b) => parseInt(a) - parseInt(b));


        } catch (error) {
            console.error('请求失败:', error);
        }
    }


    onMounted(() => {
        fetchMainData();
    });

  </script>
  
  <style scoped>
    .tag {
        cursor: pointer;
        margin-left: 10px; 
        margin-top: 10px; 
        font-size: 20px;
    }

    .rounded-border {
        display: inline-block;        /* 使 span 元素可以设置宽高和边框 */
        padding: 2px 10px;            /* 内边距，确保内容不会紧贴边框 */
        border: 1px solid gray;       /* 边框样式，可以设置颜色和粗细 */
        border-radius: 10px;          /* 圆角半径，调整数值以控制圆角的程度 */
        margin: 5px;                  /* 外边距，使元素之间有间隔 */
        font-size: 16px;              /* 字体大小 */
    }



    table {
        width: 100%;
        border-collapse: collapse;
        table-layout: fixed; 
    }

    th, td {
        border: 1px solid #ddd;
        padding: 2px;
        word-wrap: break-word; 
    }

    th {
        background-color: #e0e0e0;
        font-weight: bold;
    }

  </style>