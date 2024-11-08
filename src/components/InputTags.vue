<!-- 输入框可输入多个标签内容  -->
<template>
  <div
    @click="focusNewTag()"
    :class="{
      'read-only': readOnly,
      'vue-input-tag-wrapper--active': state.isInputActive,
    }"
    class="vue-input-tag-wrapper"
  >
    <draggable
      v-model="inputTags"
      item-key="index"
      animation="300"
      :disabled="!props.draggable"
      @update="draggableUpdate"
    >
      <template #item="{ element, index }">
        <span :key="index" class="input-tag-item text-ellipsis">
          <!-- 标签内容 -->
          <span class="tag-text">{{ element }}</span>
          <!-- 关闭标签图标 -->
          <span
            v-if="!readOnly"
            @click.prevent.stop="remove(index)"
            class="tag-close"
          >
            <el-icon v-if="!hasRemoveIconSlot"><Close /></el-icon>
            <slot name="remove-icon" />
          </span>
        </span>
      </template>
    </draggable>
    <input
      ref="inputRef"
      v-if="!readOnly && !isLimit"
      :placeholder="placeholder"
      v-model="state.newTag"
      @keydown.delete.stop="removeLastTag"
      @keydown.stop="addNew"
      @blur="handleInputBlur"
      @focus="handleInputFocus"
      class="input-div"
    />
  </div>
</template>

<script setup lang="ts">
import _ from 'lodash'
import { onMounted, ref, reactive, computed, PropType, useSlots } from 'vue'
import { Close } from '@element-plus/icons-vue'
import { ElIcon, ElMessage } from 'element-plus'
import draggable from 'vuedraggable'

const inputRef = ref()

type ObjType = {
  [key: string]: any
}
const slots = useSlots()
const hasRemoveIconSlot = !!slots['remove-icon']
type ValidateType = string | ((...args: any[]) => any) | Record<string, any>
const modelValue = defineModel()
const props = defineProps({
  // 传入的数组内容
  value: {
    type: Array,
    default: () => [],
  },
  placeholder: {
    type: String,
    default: '请输入，按回车键确定',
  },
  // 只读，不可添加不可删除
  readOnly: {
    type: Boolean,
    default: false,
  },
  // 添加标签的快捷键
  addTagWays: {
    type: Array,
    default: function () {
      return ['Enter', 'Tab']
    },
  },
  validate: {
    type: [String, Function, Object] as PropType<ValidateType>,
    default: '',
  },
  // 是否在失去焦点时添加
  addTagOnBlur: {
    type: Boolean,
    default: false,
  },
  // 限制添加个数， -1为无限制
  limit: {
    type: Number,
    default: -1,
  },
  // 内容是否可以重复
  allowDuplicate: {
    type: Boolean,
    default: false,
  },
  // 在添加前进行规范化
  beforeAdding: {
    type: Function,
  },
  // 拖拽是否开启,默认关闭
  draggable: {
    type: Boolean,
    default: false,
  },
})
const inputTags = ref([...props.value])
const emit = defineEmits(['updateInputTags'])
const state = reactive({
  newTag: '',
  isInputActive: false,
})
const isLimit = computed(
  () => props.limit > 0 && inputTags.value?.length === Number(props.limit)
)

// 组件内置的验证：邮箱、phone
const validators: ObjType = {
  email: new RegExp(/^[^\s@]+@[^\s@]+\.[^\s@]+$/),
  phone: new RegExp(/^1[3-9]\d{9}$/),
}
onMounted(() => {
  // 初始化时触发一次更新
  updateTags()
})
function focusNewTag() {
  if (props.readOnly || !inputRef.value) {
    return
  }
  inputRef.value.focus()
}

function handleInputFocus() {
  state.isInputActive = true
}

function handleInputBlur(e: FocusEvent) {
  state.isInputActive = false
  addNew(e)
}
async function addNew(e: any) {
  // 检测当前按键是否是添加标签的快捷键，默认是 ['Enter', 'Tab']
  const keyShouldAddTag = e && props.addTagWays.indexOf(e.code) !== -1
  // 事件类型是否为 blur
  const typeIsNotBlur = e && e.type !== 'blur'
  // 不添加的条件为：1. 不是添加标签的快捷键，2. 不是 blur 事件，3. 当前标签数量已经达到限制。
  if (
    (!keyShouldAddTag && (typeIsNotBlur || !props.addTagOnBlur)) ||
    isLimit.value
  ) {
    return
  }
  // 检测是否为空
  if (_.isEmpty(state.newTag.trim())) {
    state.newTag = ''
    return ElMessage.warning('请输入内容')
  }
  // 添加前是否需要规范化
  const tag = props.beforeAdding
    ? await props.beforeAdding(state.newTag)
    : state.newTag
  // 校验
  const isValid = await validateIfNeeded(tag)
  // 如果标签有效且不重复，则将其添加到 innerTags 中，并清空输入框。
  if (
    tag &&
    isValid &&
    (props.allowDuplicate || inputTags.value?.indexOf(tag) === -1)
  ) {
    inputTags.value?.push(tag)
    state.newTag = ''
    updateTags()
  }
  e && e.preventDefault()
}

function validateIfNeeded(tagValue: string) {
  if (props.validate === '' || props.validate === undefined) {
    return true
  }

  if (typeof props.validate === 'function') {
    return props.validate(tagValue)
  }

  if (
    typeof props.validate === 'string' &&
    Object.keys(validators).indexOf(props.validate) > -1
  ) {
    return validators[props.validate].test(tagValue)
  }

  if (typeof props.validate === 'object' && props.validate.test) {
    return props.validate?.test(tagValue)
  }

  return true
}
function remove(index: number) {
  inputTags.value?.splice(index, 1)
  updateTags()
}

function removeLastTag() {
  if (state.newTag) {
    return
  }
  inputTags.value?.pop()
  updateTags()
}
// 更新内容
function updateTags() {
  emit('updateInputTags', inputTags.value)
}
// 元素位置改变时更新内容
function draggableUpdate(e: any) {
  updateTags()
}
</script>
<style scoped lang="scss">
$tag-color: var(--el-color-primary);
$tag-border-color: var(--el-color-primary-light-8);
$tag-background-color: var(--el-color-primary-light-9);

.vue-input-tag-wrapper {
  width: 100%;
  height: 100%;
  background-color: #fff;
  border: 1px solid #ccc;
  cursor: pointer;
  text-align: left;
  -webkit-appearance: textfield;
  padding: 4px;
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
  align-items: center;
  box-sizing: border-box;
}

.input-tag-item {
  height: 24px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  margin-right: 4px;
  padding: 0 9px;
  font-weight: 400;
  border: 1px solid $tag-border-color;
  border-radius: 2px;
  border-radius: 4px;
  color: $tag-color;
  background-color: $tag-background-color;
  box-sizing: border-box;
}
.tag-text {
  margin-right: 10px;
}
// 超出显示省略号
.text-ellipsis {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

//  关闭标签图标
.tag-close {
  width: 1em;
  height: 1em;
  line-height: 1em;
  font-size: 14px;
  text-align: center;
  cursor: pointer;
}
.tag-close:hover {
  text-decoration: none;
  border-radius: 50%;
  background-color: $tag-color;
  color: #fff;
}

// 输入内容
.input-div {
  color: #777;
  font-weight: 400;
  outline: none;
  border: none;
  padding: 4px;
  flex-grow: 1;
}

.vue-input-tag-wrapper.read-only {
  cursor: default;
}
</style>
