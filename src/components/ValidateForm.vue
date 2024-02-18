<template>
  <form class="validate-form-container">
    <!-- 默认插槽 -->
    <slot name="default"></slot>
    <!-- @click.prevent 阻止默认事件 -->
    <div class="submit-area" @click.prevent="submitForm">
      <!-- 具名插槽 -->
      <slot name="submit">
        <!-- 插槽默认内容 -->
        <button type="submit" class="btn btn-primary">提交</button>
      </slot>
    </div>
  </form>
</template>

<script lang="ts">
import { defineComponent, onUnmounted } from 'vue'
import mitt from 'mitt'
// 类型别名，将() => boolean 函数别名为ValidateFunc
type ValidateFunc = () => boolean
// mitt传入的事件类型与名称需要一致
type ClearFunc = () => void
type ClearStatusFunc = () => void
interface CallbackProps {
  validator: ValidateFunc;
  clearInput: ClearFunc;
  clearStatus: ClearStatusFunc;
  formName: string;
}
// mitt() 实现全局事件总线vue3版
export const emitter = mitt()
export default defineComponent({
  // emit事件
  emits: ['form-submit'],
  props: {
    // 请提供不同的 formName 当使用多个表单在同一个页面中的时候
    name: {
      type: String,
      default: 'default'
    }
  },
  setup(props, context) {
    // ValidateFunc函数数组
    let funcArr: ValidateFunc[] = []
    let clearArr: ClearFunc[] = []
    const clearStatusArr: ClearStatusFunc[] = []
    // emit事件触发的函数
    const submitForm = () => {
      // 需要先map,every在遇到错误时会直接停止，map将数组中每一项转化为函数结果，再通过every判断是否每一项都为true
      const result = funcArr.map(func => func()).every(result => result)
      context.emit('form-submit', result)
    }
    // ?为可选链运算符，在func为null或者undefined时也不会报错，相当于func&&func:ValidateFunc
    const callback = (obj?: CallbackProps) => {
      if (obj && obj.formName === props.name) {
        funcArr.push(obj.validator)
        clearArr.push(obj.clearInput)
        clearStatusArr.push(obj.clearStatus)
      }
    }
    const clearInputs = () => {
      clearArr.forEach(clear => clear())
    }
    // 添加一个新的方法，用于清空现在 inputs 的所有状态，来自 ValidateInput 的 clearStatus 方法
    const clearAllStatus = () => {
      clearStatusArr.forEach(clear => clear())
    }
    // 通过emitter.on接收消息，接收传来的validateInput函数
    emitter.on('form-item-created', callback)
    onUnmounted(() => {
      emitter.off('form-item-created', callback)
      funcArr = []
      clearArr = []
    })
    return {
      submitForm,
      clearInputs,
      clearAllStatus
    }
  }
})
</script>
