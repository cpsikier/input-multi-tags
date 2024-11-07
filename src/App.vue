<template>
  <div>
    <h3>标签输入框</h3>
    <InputTags :value="data" />
  </div>
</template>
<script setup lang="ts">
import { ref } from 'vue'
import InputTags from './components/InputTags.vue'

const data = ref(['vue', 'react', 'angular'])
</script>

<style scoped></style>
