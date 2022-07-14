<!--组件使用案例-->
<!--Usage Example-->
<template>
  <Editor
      :config-path="configPath"
      class="editor"
      v-model="content"></Editor>
</template>
<script lang="ts" setup>

/**
 * 编辑器使用的配置文件
 * Editor's Config file
 * @description 可以自己定义一份配置文件，自定义选择工具栏需要使用的工具
 * 配置文件的路径：[你的项目的根目录]/ueditor/config/ueditor.config.js
 * vue3项目请将资源文件：/public/editor复制到自己根目录的/public文件夹下
 * @description You can add another config file to build your custom editor.
 * Config File Path: [Your_Project_Base_Path]/ueditor/config/ueditor.config.js
 * Please Copy you Resource folder: /public/ueditor to you own project folder: /public/ueditor
 * **/
import { ref, watch } from 'vue'
import Editor from './Editor.vue'

const configPath = '/ueditor/config/ueditor.config.js'

// 编辑器绑定的内容
// Data Bind in Editor
const content = ref('')

watch(content, last => {
  console.log(last)
}, {
  deep: true,
  immediate: true
})
</script>

<style scoped></style>