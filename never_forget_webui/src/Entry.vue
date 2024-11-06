<template>
    <div class="container">
      <!-- 左侧菜单栏 -->
      <div class="sidebar">
        <el-menu
          class="el-menu-vertical-demo"
          :default-active="activeMenu"
          @select="handleMenuSelect"
          background-color="#545c64"
          text-color="#fff"
          active-text-color="#ffd04b"
          :default-openeds="['1', '1-7', '1-8']"
        >
          <el-sub-menu index="1">
            <template #title>
              <span>菜单</span>
            </template>
            <el-menu-item index="1-1">1.语文</el-menu-item>
            <!--
            <el-menu-item index="1-2-1">1.英语-识</el-menu-item>
            <el-menu-item index="1-2-2">1.英语-写</el-menu-item>
            -->
            <el-menu-item index="1-4">2.复习打印单</el-menu-item>
            <el-sub-menu index="1-7">
              <template #title>3.词库-语文</template>
              <el-menu-item index="1-7-1">作文</el-menu-item>
              <el-menu-item index="1-7-2">人教-五上</el-menu-item>
            </el-sub-menu>
            <!--
            <el-sub-menu index="1-8">
              <template #title>3.词库-英语</template>
              <el-menu-item index="1-8-1">沪教</el-menu-item>
              <el-menu-item index="1-8-2">NM-4A</el-menu-item>
            </el-sub-menu>
            -->
          </el-sub-menu>
          <el-menu-item index="recently_charts_cmp">5.复习历史</el-menu-item>
          <el-menu-item index="recently_plan_list_cmp">5.复习计划</el-menu-item>
          <el-menu-item index="object_analysis_cmp">6.单词熟练度</el-menu-item>
          <el-menu-item index="object_warehouse_analysis_cmp">7.课程进度-错</el-menu-item>
          <el-menu-item index="object_warehouse_analysis_rc_cmp">7.课程进度-对</el-menu-item>
          
          
        </el-menu>
      </div>
      <!-- 右侧内容区域 -->
      <div class="content">
        <component :is="currentComponent" :menuIndex="activeMenu" :key="activeMenu" :filterData="filterData"></component>
      </div>
    </div>
  </template>
  
  <script setup>
  import { shallowRef } from 'vue';
  import Component1_1 from './components/Main.vue';
  import Component1_2_1 from './components/Main_en_k.vue';
  import Component1_2_2 from './components/Main_en_w.vue';
  import Component1_3 from './components/import_object_list.vue';
  import Component1_4 from './components/get_today_review_files.vue';
  import Component1_5 from './components/import_review_result.vue';
  import Component1_6 from './components/object_warehouse.vue';
  import recently_charts_cmp from './components/recently_charts.vue';
  import recently_plan_list_cmp from './components/recently_plan_list.vue';
  import object_analysis_cmp from './components/object_analysis.vue';
  import object_warehouse_analysis_cmp from './components/object_warehouse_analysis.vue';
  import object_warehouse_analysis_rc_cmp from './components/object_warehouse_analysis_rc.vue';
  

  const activeMenu = shallowRef('1-1');
  const currentComponent = shallowRef(Component1_1);
  
  const filterData = {
    title : '',
    subject : '',
    textbook : '',
    grade : '',
    section : '',
    k : '',
    w : ''

  };

  const handleMenuSelect = (index) => {
    activeMenu.value = index;
    switch (index) {
        case '1-1':
            currentComponent.value = Component1_1;
            break;
        case '1-2-1':
            currentComponent.value = Component1_2_1;
            break;
        case '1-2-2':
            currentComponent.value = Component1_2_2;
            break;
        case '1-3':
            currentComponent.value = Component1_3;
            break;
        case '1-4':
            currentComponent.value = Component1_4;
            break;
        case '1-5':
            currentComponent.value = Component1_5;
            break;
        case 'recently_charts_cmp':
            currentComponent.value = recently_charts_cmp;
            break;
        case 'recently_plan_list_cmp':
            currentComponent.value = recently_plan_list_cmp;
            break;
        case 'object_analysis_cmp':
            currentComponent.value = object_analysis_cmp;
            break;
        case 'object_warehouse_analysis_cmp':
            currentComponent.value = object_warehouse_analysis_cmp;
            break;
        case 'object_warehouse_analysis_rc_cmp':
            currentComponent.value = object_warehouse_analysis_rc_cmp;
            break;
        case '1-7-1':
            filterData.title = '语文，作文';
            filterData.subject = '语文';
            filterData.textbook = '作文';
            filterData.grade = '';

            currentComponent.value = Component1_6;
            break;
        case '1-7-2':
            filterData.title = '语文，人教版，五年级上';
            filterData.subject = '语文';
            filterData.textbook = '人教版';
            filterData.grade = '五年级上';
            currentComponent.value = Component1_6;
            break;
        case '1-8-1':
            filterData.title = 'english，沪教';
            filterData.subject = 'english';
            filterData.textbook = '沪教';
            filterData.grade = '五年级上';
            currentComponent.value = Component1_6;
            break;
        case '1-8-2':
            filterData.title = 'english，NM，4A';
            filterData.subject = 'english';
            filterData.textbook = 'NM';
            filterData.grade = '4A';
            currentComponent.value = Component1_6;
            break;
        default:
            currentComponent.value = Component1_1;
    }
  };
  </script>
  
  <style scoped>
  .container {
    display: flex;
    height: 100vh;
  }
  
  .sidebar {
    width: 180px;
    height: 100%; /* 让菜单占满容器 */
    overflow-y: auto; /* 让菜单可以滚动 */
    background-color: #dbdde0;
  }
  
  .content {
    flex: 1;
    padding: 0;
    overflow: auto;
  }
  </style>
  