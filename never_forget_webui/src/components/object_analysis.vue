<template>
    <div>

        <table >
            <thead>
                <tr>
                    <th style="width:100px;text-align: center;">熟练度</th>
                    <th style="width:60px;text-align: center;">个数</th>
                    <th style="text-align: center;">单词</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="index in totalReviewCount" :key="index" > 

                    <td style="text-align: center;"> 
                       <span style="font-size: 24px;font-weight: bold;  "> {{ index }} </span>
                    </td>
                    <td style="text-align: center;"> 
                       <span style="font-size: 16px;color: red; "> {{ getObjectCountByReviewCount(index) }} 个 </span>
                    </td>
                    <td> 
                        <div style="margin-top: 10px;margin-bottom: 10px;">
                            <span v-for="obj in getObjectsByReviewCount(index)" :key="obj.id"  class="rounded-border" :style="getSpanStyle(obj)">
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

    const totalReviewCount = ref([1,2,3,4,5,6,7,8,9,10,11]);


    const mainData = ref([]);
    const reviewType = ref('CN');

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

    const getObjectCountByReviewCount = (review_count) => {
        return mainData.value.filter(item => item.review_count === review_count).length;
    };

    const getObjectsByReviewCount = (review_count) => {
        let data = mainData.value.filter(item => item.review_count === review_count);

        //按照wrong_count排序
        data.sort((a, b) => b.wrong_count - a.wrong_count);
        return data;
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
            

            let url = 'http://0.0.0.0:30088/get_all_object_list_';

            if (reviewType.value === 'CN') {
                url += 'cn_w';
            } else if (reviewType.value === 'ENK') {
                url += 'english_k';
            } else if (reviewType.value === 'ENW') {
                url += 'english_w';
            }

            const response = await axios.get(url, {
                }, {
                    headers: {
                        'accept': 'application/json',
                        'Content-Type': 'application/json'
                    }
                });
            mainData.value = response.data.data_list;


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