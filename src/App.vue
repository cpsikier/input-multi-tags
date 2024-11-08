<template>
  <div>
    <h3>标签输入框</h3>
    <InputTags :value="data" draggable @updateInputTags="getInputTags" />
    <el-divider></el-divider>
    原始内容：{{ data }}
    <el-divider></el-divider>
    输入框内容：{{ inputValue }}
  </div>
</template>
<script setup lang="ts">
import { ref } from 'vue'
import { ElDivider } from 'element-plus'
import InputTags from './components/InputTags.vue'

const data = ref(['vue', 'react', 'angular'])
const inputValue = ref()
function getInputTags(val: string[]) {
  inputValue.value = val
  console.log(`[log] - updateInput - updateInput:`, val)
}
</script>

<style scoped></style>
