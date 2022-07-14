<template>
  <div>
    <editor
        :config-path="configPath"
        class="editor"
        v-model="content"></editor>
  </div>
</template>

<script lang="ts" setup>
import { reactive, ref, computed, watch } from 'vue'
import { Example } from './dev.json'

// 编辑器使用的配置文件
const configPath = '/ueditor/contractConfig/contract.editor.config.js'
const content = ref(Example)

watch(content, last => {
  console.log(`输入内容：${last}`)
}, {
  deep: true,
  immediate: true
})
</script>

<style scoped>

</style>
