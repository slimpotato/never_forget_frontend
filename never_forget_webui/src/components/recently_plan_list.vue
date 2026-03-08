<template>
    <div>
        
        <table >
            <thead>
                <tr>
                    <th style="width:150px;text-align: center;">日期</th>
                    <th style="width:60px;text-align: center;">个数</th>
                    <th style="text-align: center;">单词</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="dateStr in dateStrList" :key="dateStr" > 

                    <td style="text-align: center;"> 
                       <span style="font-size: 24px;font-weight: bold;  "> {{ dateStr }} </span><br/>
                       <span :style="isWeekendStyle(dateStr)"> （ {{ getWeekDayByDateStr(dateStr) }} ） </span>
                    </td>
                    <td style="text-align: center;"> 
                       <span style="font-size: 16px;font-weight: bold; color: red; "> {{ getObjectCountByDateStr(dateStr) }} 个 </span>
                    </td>
                    <td> 
                        <div style="margin-top: 10px;margin-bottom: 10px;">
                            <span v-for="obj in getObjectsByDateStr(dateStr)" :key="obj.id"  class="rounded-border" :style="getSpanStyle(obj)">
                            <el-tooltip :content="getTooltipContent(obj)" placement="top" effect="light">
                                <span >
                                   <span style="font-size: 16px; font-weight: bold;"> {{ obj.object_content }} </span> 
                                   <span style="font-size: 10px; color: gray;">✓{{ obj.review_count }}</span>
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

    const mainData = ref([]);
    const dateStrList = ref([]);

    //根据日期字符串，获得中文的星期一、星期二、星期三等，注意：星期一是一周的开始
    const getWeekDayByDateStr = (dateStr) => {
        const weekDays = ["星期日", "星期一", "星期二", "星期三", "星期四", "星期五", "星期六"];

        // 获取当前日期
        const today = new Date(dateStr);

        // 获取星期几的数字（0 表示星期日，6 表示星期六）
        const dayOfWeek = today.getDay();

        // 获取对应的中文星期
        const dayName = weekDays[dayOfWeek];

        return dayName;
    };

    const isWeekendStyle = (dateStr) => {
        const dayOfWeek = new Date(dateStr).getDay();
        const result = {}
        if(dayOfWeek === 0 || dayOfWeek === 6){
            result.color = 'red';
            result.fontWeight = 'bold';
            result.fontSize = '16px';
        } else {
            result.color = 'black';
            result.fontWeight = 'normal';
            result.fontSize = '16px';
        }

        return result;
    };

    const getTooltipContent = (obj) => {
        return `${obj.object_reminder} | 
         ${obj.review_day_descp} | 
         对 ${obj.review_count} 次 `;
    };


    const getSpanStyle = (obj) => {

        return {
            backgroundColor: getBGColorByReviewCount(obj.review_count),
            color:  'black'
        };
    };

    const getLocaleDateString = () => {
        const date = new Date();
        const year = date.getFullYear();
        const month = String(date.getMonth() + 1).padStart(2, '0');
        const day = String(date.getDate()).padStart(2, '0');
        return `${year}-${month}-${day}`;
    };

    const todayStr = ref(getLocaleDateString());


    const getBGColorByReviewCount = (value) => {
        const colors = [
            '#FFFF00', // 黄色
            '#F2FF00',
            '#E5FF00',
            '#CCFF00',
            '#99FF00',
            '#66FF00',
            '#33FF00',
            '#00FF00'  // 绿色
        ];

        if (value == null || value <= 1) {
            return colors[0]; // 当 value 为 null、0 或 1 时，返回黄色
        }

        return colors[Math.min(value - 1, colors.length - 1)]; // 返回对应的颜色，最多为绿色
    };

    const getObjectsByDateStr = (dateStr) => {

        let result = [];
        //如果是今天，则需要把今天以前的日期对应的object也计算在内
        if (dateStr === todayStr.value) {
            result = mainData.value.filter(item => item.next_review_time.split(' ')[0] <= dateStr);
        } else {
            result = mainData.value.filter(item => item.next_review_time.split(' ')[0] === dateStr);
        }

        //根据review_count排序
        result.sort((a, b) => a.review_count - b.review_count);

        return result;
        
    };

    const getObjectCountByDateStr = (dateStr) => {
        //如果是今天，则需要把今天以前的日期对应的object也计算在内
        if (dateStr === todayStr.value) {
            return mainData.value.filter(item => item.next_review_time.split(' ')[0] <= dateStr).length;
        } else {
            return mainData.value.filter(item => item.next_review_time.split(' ')[0] === dateStr).length;
        }
    };

    const fetchMainData = async () => {
        try {

            mainData.value = [];
            

            let url = 'http://0.0.0.0:30088/get_all_object_list_cn_w';

            const response = await axios.get(url, {
                }, {
                    headers: {
                        'accept': 'application/json',
                        'Content-Type': 'application/json'
                    }
                });
            mainData.value = response.data.data_list;

            //根据reviewtime 生成dateStrList
            dateStrList.value = mainData.value.map(item => item.next_review_time.split(' ')[0]);

            //去除重复，并排序
            dateStrList.value = [...new Set(dateStrList.value)].sort();

            //去除今天以前的日期，保留今天的日期
            dateStrList.value = dateStrList.value.filter(dateStr => {
                return dateStr >= todayStr.value;
            });

            console.log(dateStrList.value);
            

        } catch (error) {
            console.error('Error fetching main data:', error);
        }
    };

    onMounted(() => {
        fetchMainData();
    });

</script>

<style scoped>

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