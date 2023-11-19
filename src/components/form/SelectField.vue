<template>
  <div class="select-field">
    <div class="wrapper">
      <div @click="isActive = !isActive" class="current">
        <span :class="props.modelValue">{{ props.modelValue }}</span>
        <IconAngleDown />
      </div>
      <ul v-if="isActive" class="items">
        <li
          v-for="item in items"
          :key="item.value"
          @click="updateModelValue(item.value)"
          :class="item.value"
        >
          {{ item.title }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref, defineProps, onBeforeUnmount } from 'vue'
import IconAngleDown from '../icons/IconAngleDown.vue'

const props = defineProps({
  modelValue: String,
  items: Array
})

const isActive = ref(false)

const emit = defineEmits(['update:modelValue'])

const updateModelValue = (value) => {
  emit('update:modelValue', value)
  isActive.value = false
}

const onDocumentClick = (event) => {
  if (!document.querySelector('.wrapper').contains(event.target)) {
    isActive.value = false
  }
}
document.addEventListener('click', onDocumentClick)

onBeforeUnmount(() => {
  document.removeEventListener('click', onDocumentClick)
})
</script>

<style lang="scss" scoped>
.select-field {
  z-index: 1;
  .wrapper {
    position: relative;
    .current {
      display: flex;
      align-items: center;
      justify-content: space-between;
      width: 110px;
      padding: 10px;
      cursor: pointer;
      span {
        text-transform: uppercase;
      }
      svg {
        height: 10px;
        fill: var(--color-text);
      }
    }
    .items {
      display: flex;
      flex-direction: column;
      gap: 5px;
      list-style: none;
      padding: 10px;
      width: 150px;
      background: var(--color-background);
      border: solid 2px var(--color-border);
      position: absolute;
      left: 0;
      top: 100%;
      li {
        padding: 5px 10px;
        text-transform: uppercase;
        cursor: pointer;
        border-radius: 5px;
        &:hover {
          background: #262626;
        }
      }
    }
  }
}
</style>