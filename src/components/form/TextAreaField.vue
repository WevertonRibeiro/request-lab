<template>
  <div class="textarea-wrapper">
    <textarea
      @keydown.tab.prevent="handleTabKeyPress"
      @input="updateModelValue"
      :name="props.name"
      :id="props.id"
      :value="props.modelValue"
    ></textarea>
  </div>
</template>

<script setup>
import { defineProps } from 'vue'

const emit = defineEmits(['update:modelValue'])

const props = defineProps({
  modelValue: String,
  name: String,
  id: String
})

const handleTabKeyPress = (event) => {
  if (event.keyCode === 9) {
    event.preventDefault()

    const textarea = event.target
    const cursorPosition = textarea.selectionStart

    textarea.value =
      textarea.value.substring(0, cursorPosition) +
      '    ' +
      textarea.value.substring(cursorPosition)

    window.getSelection().removeAllRanges()
    textarea.selectionStart = cursorPosition + 4
    emit('update:modelValue', textarea.value)
  }
}

const updateModelValue = (event) => {
  emit('update:modelValue', event.target.value)
}
</script>

<style lang="scss" scoped>
.textarea-wrapper {
  position: relative;
  width: 100%;
  height: 100%;
  &::before {
    content: '';
    display: block;
    width: 17px;
    height: 17px;
    position: absolute;
    right: 0;
    bottom: 0;
    background: var(--color-background);
    border-right: solid 2px var(--color-border);
    border-bottom: solid 2px var(--color-border);
    border-radius: 0 0 5px 0;
    z-index: 1;
  }
  textarea {
    width: 100%;
    height: 100%;
    padding: 15px;
    border: solid 2px var(--color-border);
    background: var(--color-background);
    outline: none;
    color: #ce9178;
    border-radius: 5px;
  }
}
</style>