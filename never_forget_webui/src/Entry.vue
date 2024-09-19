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
        >
          <el-sub-menu index="1">
            <template #title>
              <span>菜单</span>
            </template>
            <el-menu-item index="1-1">全部生词(语文)</el-menu-item>
            <el-menu-item index="1-2">全部生词(english)</el-menu-item>
            <el-menu-item index="1-3">1.录入生词</el-menu-item>
            <el-menu-item index="1-4">2.生成复习打印单</el-menu-item>
            <el-menu-item index="1-5">3.导入复习结果</el-menu-item>
          </el-sub-menu>
        </el-menu>
      </div>
      <!-- 右侧内容区域 -->
      <div class="content">
        <component :is="currentComponent"></component>
      </div>
    </div>
  </template>
  
  <script setup>
  import { shallowRef } from 'vue';
  import Component1_1 from './components/Main.vue';
  import Component1_2 from './components/Main_en.vue';
  import Component1_3 from './components/import_object_list.vue';
  import Component1_4 from './components/get_today_review_files.vue';
  import Component1_5 from './components/import_review_result.vue';
  
  const activeMenu = shallowRef('1-1');
  const currentComponent = shallowRef(Component1_1);
  
  const handleMenuSelect = (index) => {
    activeMenu.value = index;
    switch (index) {
        case '1-1':
            currentComponent.value = Component1_1;
            break;
        case '1-2':
            currentComponent.value = Component1_2;
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
    background-color: #dbdde0;
  }
  
  .content {
    flex: 1;
    padding: 0;
    overflow: auto;
  }
  </style>
  