<script setup>
// components
import FakeItem from '@/components/FakeItem.vue'

// base
import { computed, nextTick, reactive, ref, watch } from 'vue'
import _ from 'lodash'

// props
const props = defineProps({
  dataList: {
    type: Array,
    required: true
  }
})

// data
const itemMinHeight = parseFloat(import.meta.env.VITE_APP_ITEM_MIN_HEIGHT)
const wrap = ref()
const info = reactive({
  visibleHeight: 0,
  visibleCount: 0,
  startIndex: 0,
  endIndex: 0,
  offset: 0
})

// buffer
const BUFFER_SCALE = 1
const bufferCount = computed(() => Math.floor(info.visibleCount * BUFFER_SCALE))
const visibleStartIndex = computed(() => info.startIndex - Math.min(info.startIndex, bufferCount.value))
const visibleEndIndex = computed(() => info.endIndex + Math.min(props.dataList.length - info.endIndex, bufferCount.value))

// visible list
const visibleList = computed(() => {
  return props.dataList
    .slice(visibleStartIndex.value, visibleEndIndex.value)
    .map((data, index) => ({ index: visibleStartIndex.value + index, item: data }))
})

// cache
const cacheList = ref([])
watch(
  () => props.dataList.length,
  (newValue, oldValue) => {
    // default
    const isAdd = newValue - oldValue > 0
    while (props.dataList.length !== cacheList.value.length) {
      if (isAdd) {
        const prevItem = cacheList.value.at(-1)
        const prevItemHeight = prevItem?.height ?? 0
        const prevItemTop = prevItem?.top ?? 0
        cacheList.value.push({
          height: itemMinHeight,
          top: prevItemHeight + prevItemTop
        })
      } else {
        cacheList.value.pop()
      }
    }
    // Init Info
    nextTick(() => {
      info.visibleHeight = wrap.value?.clientHeight ?? 0
      info.visibleCount = Math.ceil(info.visibleHeight / itemMinHeight)
      info.end = info.startIndex + info.visibleCount
    })
  }
)

// update dom
function onUpdateCache(el, index) {
  if (!el) return

  const cacheItem = cacheList.value[index]
  const newHeight = el.getBoundingClientRect().height
  const diffHeight = newHeight - cacheItem.height
  if (diffHeight === 0) return

  cacheItem.height = newHeight
  for (let j = index + 1; j < cacheList.value.length; j++) {
    cacheList.value[j].top += diffHeight
  }
}

// total height
const totalHeight = computed(() => {
  const lastCache = cacheList.value.at(-1)
  return lastCache ? `${lastCache.top + lastCache.height}px` : 'auto'
})

// scroll
function onScroll(e) {
  const target = e.target
  if (!target) return

  const scrollTop = target.scrollTop
  debounceUpdateInfo(scrollTop)
  throttleUpdateInfo(scrollTop)
}
const debounceUpdateInfo = _.debounce(updateInfo, 240)
const throttleUpdateInfo = _.throttle(updateInfo, 80)
function updateInfo(scrollTop) {
  info.startIndex = getStartIndex(scrollTop)
  info.endIndex = info.startIndex + info.visibleCount
  info.offset = cacheList.value[visibleStartIndex.value].top
}
function getStartIndex(scrollTop) {
  let left = 0
  let right = cacheList.value.length - 1
  while (left !== right) {
    const mid = Math.ceil((left + right) / 2)
    const midTop = cacheList.value[mid].top
    const midBottom = midTop + cacheList.value[mid].height
    if (scrollTop >= midTop && scrollTop <= midBottom) {
      return mid
    } else if (scrollTop < midTop) {
      right = mid - 1
    } else {
      left = mid + 1
    }
  }

  const lastItem = cacheList.value[left]
  const lastTop = lastItem.top
  const lastBottom = lastTop + lastItem.height
  return scrollTop > lastItem.top && scrollTop < lastBottom ? left : 0
}
</script>

<template>
  <div class="container">
    <div v-if="dataList.length > 0" ref="wrap" class="scroll-wrap" @scroll="onScroll">
      <div :style="{ height: totalHeight }">
        <div
          v-for="data in visibleList"
          :key="data.item.id"
          :ref="(el) => onUpdateCache(el, data.index)"
          class="scroll-item"
          :class="data.index % 2 === 0 ? 'scroll-item-even' : 'scroll-item-odd'"
          :style="{ transform: `translateY(${info.offset}px)` }"
        >
          <FakeItem>{{ data.item.title }}</FakeItem>
        </div>
      </div>
    </div>
    <div v-else>No Data</div>
  </div>
</template>

<style>
.container {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1rem;
}

.scroll-wrap {
  width: 75%;
  max-height: 70vh;
  border: 1px solid #333333;
  overflow: auto;
}

.scroll-item {
  font-size: 1.5rem;
  font-weight: bold;
  padding: 0 1rem;
}
.scroll-item-odd {
  background-color: rgba(0, 0, 0, 0.2);
}
.scroll-item-even {
  background-color: rgba(0, 0, 0, 0.1);
}
</style>
