<template>
  <div class="vue-easymde-editor">
    <textarea ref="textArea"></textarea>
  </div>
</template>
<script lang="ts" setup>
// setup语法糖
import { ref, onMounted, onUnmounted, watch } from 'vue'
import EasyMDE, { Options } from 'easymde'
// 类型，属性以及事件
interface EditorProps {
  modelValue?: string;
  options?: Options;
}
interface EditorEvents {
  (type: 'update:modelValue', value: string): void;
  (type: 'change', value: string): void;
  (type: 'blur'): void;
}
const props = defineProps<EditorProps>()
const emit = defineEmits<EditorEvents>()
// 有了模版我们需要一些初始的数据
// 1 暴露对应的方法
// 2 结合页面实现验证功能
// 获取对应的节点
const textArea = ref<null | HTMLTextAreaElement>(null)
let easyMDEInstance: EasyMDE | null = null
// 双向绑定的文本框内容
const innerValue = ref(props.modelValue || '')
// 编辑文章时，通过axios异步返回父组件传来的文本内容，需要通过watch监测，否则无法更新值
watch(() => props.modelValue, (newValue) => {
  if (easyMDEInstance) {
    if (newValue !== innerValue.value) {
      // 为easyMDE文本框内容更新值
      easyMDEInstance.value(newValue || '')
    }
  }
})
onMounted(() => {
  if (textArea.value) {
    // 组装 options
    const config: Options = {
      ...(props.options || {}),
      element: textArea.value,
      initialValue: innerValue.value
    }
    // 创建easyMDE实例
    easyMDEInstance = new EasyMDE(config)
    // 监控对应的事件
    easyMDEInstance.codemirror.on('change', () => {
      if (easyMDEInstance) {
        // 拿到当前的值
        const updatedValue = easyMDEInstance.value()
        innerValue.value = updatedValue
        // v-models双向绑定对应的函数，更新父组件双向绑定的值
        emit('update:modelValue', updatedValue)
        // 表单基本事件
        emit('change', updatedValue)
      }
    })
    easyMDEInstance.codemirror.on('blur', () => {
      if (easyMDEInstance) {
        emit('blur')
      }
    })
  }
})
// 销毁对应的实例
onUnmounted(() => {
  if (easyMDEInstance) {
    easyMDEInstance.cleanup()
  }
  easyMDEInstance = null
})
const clear = () => {
  if (easyMDEInstance) {
    easyMDEInstance.value('')
  }
}
const getMDEInstance = () => {
  return easyMDEInstance
}
// expose（函数）暴露子组件属性或方法，父组件获取子组件实例后可以直接使用
defineExpose({
  clear,
  getMDEInstance
})
</script>
<style>
.vue-easymde-editor.is-invalid {
  border: 1px solid #dc3545;
}
</style>
