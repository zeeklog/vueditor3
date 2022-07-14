# vueditor3

这是一个支持 vue3.x 的 富文本编辑器， 基于UEditor开发。
支持UEditor所有的指令和功能，并且完美支持UEditor的自定义指令。

This is a rich text editor that supports vue3.x, developed based on UEditor.
Supports all UEditor commands and functions, and perfectly supports UEditor's custom commands.

[![NPM license](https://img.shields.io/npm/l/yaml-eslint-parser.svg)](https://github.com/ethwillupto10000/vueditor3)
[![NPM version](https://img.shields.io/npm/v/yaml-eslint-parser.svg)](https://github.com/ethwillupto10000/vueditor3)

## 安装指南 / Installation

- 一、克隆代码 / Clone Code
```bash 
git clone https://github.com/ethwillupto10000/vueditor3
cd vueditor3
npm i # 或者 cnpm i 
```

- 二、复制资源文件 / Copy Resource Folder
    - 复制文件夹：/public.ueditor 到你的项目根目录/public下（没有则创建一个）
    - Copy Folder: /public.ueditor and place it under your project root path: /public（if don't have , plz make a new 1）

## 运行开发环境 / Run dev environment
```shell
npm run dev
```
## 组件使用方法 / Usage
```vue
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
```
### Configuration

   #### 1、Copy Resource Folder to your Own Project
      - File Path: /public/ueditor
   #### 2、You can just use Editor.vue as your Own Component.

