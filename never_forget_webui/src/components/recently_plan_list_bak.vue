<template>
    <div>

        <div style="text-align: center;margin-top: 10px;">
            <el-radio-group v-model="reviewType" size="large" @change="reviewTypeChange">
                <el-radio-button label="语文" value="CN" />
                <el-radio-button label="英语-识" value="ENK" />
                <el-radio-button label="英语-写" value="ENW" />
            </el-radio-group>

            &nbsp;&nbsp;&nbsp;&nbsp;

            <el-switch
                v-model="isShowWrong"
                active-text="错误数✘"
                inactive-text="正确数✓"
                active-color="#13ce66"
                inactive-color="#ff4949"
            ></el-switch>

        </div>

        <div v-for="item in reviewDayDescpList" :key="item.review_day_descp" >
             <br/> <span> {{ item.review_day_descp }} - {{ item.next_review_time }}  
                <span style="color:red; ">（ {{ item.count }} ）</span>  </span> <br/>
            <div  >
                <span v-for="obj in getObjecsByDayDescp(item.review_day_descp)" :key="obj.id">
                   <!-- <el-badge :value="obj.review_count"  color="lightgray" :offset="[0, 15]">-->
                    <el-popover
                        placement="top-start"
                        :title="obj.object_content"
                        :width="200"
                        trigger="click"
                    >
                        <template #reference>
                            
                            
                            <el-tag type="primary" size="large" class="tag" :color="getBGColorByReviewCount(obj.review_count)" >
                                {{ obj.object_content }} 
                                <template v-if="isShowWrong">
                                    ✘{{ obj.wrong_count }}
                                </template>
                                <template v-else>
                                    ✓{{ obj.review_count }}
                                </template>
                            </el-tag> 
                            
                         </template>
                         <template #default>
                            <div>
                                <p><span style="font-weight: bold;"> 提示：</span> {{ obj.object_reminder }} </p>
                                <p><span style="font-weight: bold;"> 下次时间：</span>{{ obj.next_review_time }} </p>
                                <p><span style="font-weight: bold;"> 下次：</span>{{ obj.review_day_descp }} </p>
                                <p><span style="font-weight: bold;"> 对：</span>{{ obj.review_count }} </p>
                                <p><span style="font-weight: bold;"> 错：</span>{{ obj.wrong_count }} </p>
                                <p><span style="font-weight: bold;"> 教材：</span>{{ obj.textbook }} </p>
                                <p><span style="font-weight: bold;"> 年级：</span>{{ obj.grade }} </p>
                                <p><span style="font-weight: bold;"> 章节：</span>{{ obj.section }} </p>
                            </div>
                         </template>
                    </el-popover>
                    <!-- </el-badge>-->
                </span>
            </div>
        </div>
    </div>
</template>
  
<script setup>
    import { ref, onMounted } from 'vue';
    import axios from 'axios';

    const mainData = ref([]);
    const reviewDayDescpList = ref([]);
    const reviewType = ref('CN');

    const isShowWrong = ref(false);


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

    const reviewTypeChange = (newValue) => {
        reviewType.value = newValue;
        fetchMainData();
    };

  
    const fetchMainData = async () => {
        try {

            reviewDayDescpList.value = [];
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

            //获取所有review_day_descp和next_review_time，形成一个新的对象数组
            
            let reviewDayDescpListAll = mainData.value.map(item => ({
                review_day_descp: item.review_day_descp,
                next_review_time: new Date(item.next_review_time).toISOString().split('T')[0] 
            }));


            //去除review_day_descp==”今天“的记录，然后基于review_day_descp+next_review_time的日期去重,并且按照next_review_time的日期部分排序，并且计算每个review_day_descp的数量
            let reviewDayDescpListNoToday = reviewDayDescpListAll
                .filter(item => new Date(item.next_review_time) > new Date().setHours(23, 59, 59, 999))
                .reduce((acc, current) => {
                    const x = acc.find(item => item.review_day_descp === current.review_day_descp && item.next_review_time === current.next_review_time);
                    if (!x) {
                        current.count = 1;
                        return acc.concat([current]);
                    } else {
                        x.count += 1;
                        return acc;
                    }
                }, [])
                .sort((a, b) => new Date(a.next_review_time) - new Date(b.next_review_time));
            
            //添加“今天”，并且next_review_time为今天的日期，比如：2024-06-20
            const today = new Date().toISOString().split('T')[0];
            reviewDayDescpList.value.push({ 
                review_day_descp: '今天', 
                next_review_time: today,
                count: reviewDayDescpListAll.filter(item => new Date(item.next_review_time) <= new Date().setHours(23, 59, 59, 999)).length
             });

            //把reviewDayDescpListNoToday的数据添加到reviewDayDescpList中
            reviewDayDescpList.value = reviewDayDescpList.value.concat(reviewDayDescpListNoToday);
            
            

        } catch (error) {
            console.error('请求失败:', error);
        }
    }

    const getObjecsByDayDescp = (review_day_descp) => {
        //如果是”今天“，则返回mainData.value中next_review_time小于等于今天的数据

        let result = [];

        if (review_day_descp === '今天') {
            result = mainData.value.filter(item => new Date(item.next_review_time) <= new Date().setHours(23, 59, 59, 999));
        }
        else {
            result = mainData.value.filter(item => item.review_day_descp=== review_day_descp);
        }

        //基于review_count排序
        result.sort((a, b) => a.review_count - b.review_count);

        return result;
    };

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

  </style>