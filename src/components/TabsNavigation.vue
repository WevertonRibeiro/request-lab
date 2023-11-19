<template>
  <div class="tabs-wrapper">
    <div class="tabs">
      <div
        class="tab"
        v-for="(tab, index) in props.items"
        :key="tab.key"
        :class="{ active: tabActive === index }"
        @click="tabActive = index"
      >
        <span>{{ tab.label }}</span>
      </div>
    </div>
    <div class="contents">
      <slot :name="props.items[tabActive].key"></slot>
    </div>
  </div>
</template>

<script setup>
import { ref, defineProps } from 'vue'

const props = defineProps({
  items: Array
})

const tabActive = ref(0)
</script>

<style lang="scss" scoped>
.tabs-wrapper {
  display: flex;
  flex-direction: column;
  height: 100%;
  .tabs {
    display: flex;
    gap: 20px;
    .tab {
      cursor: pointer;
      padding: 5px 0;
      &.active {
        border-bottom: solid 2px tomato;
        span {
          color: white;
        }
      }
      span {
        font-size: 12px;
        color: var(--color-text);
      }
      &:hover {
        span {
          color: white;
        }
      }
    }
  }
  .contents {
    flex: 1;
  }
}
</style>