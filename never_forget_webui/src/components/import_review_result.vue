<template>
    <div class="container">
      导入复习结果：
      <el-upload
        ref="upload"
        class="upload-demo"
        action="http://0.0.0.0:30088/import_review_result"
        :limit="1"
        :on-exceed="handleExceed"
        :auto-upload="false"
        :on-change="handleChange"
        :on-success="on_success"

      >
        <template #trigger>
          <el-button type="primary">选择文件</el-button> &nbsp;&nbsp;&nbsp;&nbsp;
        </template>
        <el-button v-if="fileSelected" class="ml-3" type="success" @click="submitUpload">
          导入复习结果
        </el-button>
      </el-upload>
    </div>
  </template>
  
  <script setup lang="ts">
  import { ref } from 'vue'
  import { genFileId , ElMessage} from 'element-plus'
  import type { UploadInstance, UploadProps, UploadRawFile } from 'element-plus'
  
  const upload = ref<UploadInstance>()
  const fileSelected = ref(false)
  
  const handleExceed: UploadProps['onExceed'] = (files) => {
    upload.value!.clearFiles()
    const file = files[0] as UploadRawFile
    file.uid = genFileId()
    upload.value!.handleStart(file)
  }
  
  const submitUpload = () => {
    upload.value!.submit()
  }

  const on_success: UploadProps['onSuccess'] = (response) => {
        ElMessage({
        showClose: true,
        message: `导入成功，影响记录数： ${response.affect_count}`, // 假设response中有一个message字段
        type: 'success',
        duration: 10000
        });
    }


    const handleChange: UploadProps['onChange'] = (file, fileList) => {
    fileSelected.value = fileList.length > 0
    }

  </script>
  
  <style scoped>
  .container {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
  }
  
  </style>