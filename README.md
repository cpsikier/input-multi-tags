# input-multi-tags

A tag input box component based on Vue3

## Introduction

一个基于 vue3 的标签输入框组件，可在输入框中添加多个内容，可灵活配置添加方式、数量、是否只读、是否可重复添加、校验规则。

支持的功能：

- [x] 添加
- [x] 删除
- [ ] 修改
- [x] 拖拽移动

## Usage

```
<template>
  <div>
    <InputTags :value="data" draggable @updateInputTags="getInputTags" />
  </div>
</template>
```

```
<script setup lang="ts">
import { ref } from 'vue'
import InputTags from './components/InputTags.vue'

const data = ref(['vue', 'react', 'angular'])
const inputValue = ref()

// 获取更新后输入的值
function getInputTags(val: string[]) {
  inputValue.value = val
  console.log(`[log] - updateInput - updateInput:`, val)
}
</script>
```

## Attributes

| 属性名         | 说明                 | 类型                       | 默认值                 |
| -------------- | -------------------- | -------------------------- | ---------------------- |
| value          | 传入的内容           | Array                      | []                     |
| placeholder    | 输入框占位文本       | String                     | "请输入，按回车键确定" |
| readOnly       | 是否只读             | Boolean                    | false                  |
| addTagWays     | 添加标签的快捷键     | Array                      | ['Enter', 'Tab']       |
| validate       | 校验规则             | [String, Function, Object] | ''                     |
| addTagOnBlur   | 是否在失去焦点时添加 | Boolean                    | false                  |
| limit          | 限制添加的个数       | Number                     | -1 (不限制)            |
| allowDuplicate | 添加的内容是否可重复 | Boolean                    | false                  |
| beforeAdding   | 在添加前进行规范化   | Function                   |                        |
| draggable      | 是否开启拖拽功能     | Boolean                    | false                  |

## Expose

| 方法名          | 说明                                    |
| --------------- | --------------------------------------- |
| updateInputTags | 更新输入框内容,将输入框的内容传给父组件 |
