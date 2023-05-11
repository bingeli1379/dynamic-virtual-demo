<script setup>
// components
import VirtualScrollList from '@/components/VirtualScrollList.vue'

// base
import { ref } from 'vue'

// id count
let idCount = 0

// dataList
const dataList = ref([])

// functions
const BATCH_QUANTITY = 50
function addItem() {
  dataList.value.push(getItem())
}
function removeItem() {
  dataList.value.pop()
}
function addItems() {
  for (let index = 0; index < BATCH_QUANTITY; index++) {
    addItem()
  }
}
function removeItems() {
  for (let index = 0; index < BATCH_QUANTITY; index++) {
    removeItem()
  }
}
function getItem() {
  const id = ++idCount
  return {
    id,
    title: `My id  is ${id}`
  }
}
</script>

<template>
  <main>
    <h1 class="title">Dynamic Virtual Scroll</h1>

    <div class="functions">
      <button @click="addItem">add 1 item</button>
      <button @click="removeItem">remove 1 item</button>
      <button @click="addItems">add 50 item</button>
      <button @click="removeItems">remove 50 item</button>
    </div>

    <VirtualScrollList :data-list="dataList" />
  </main>
</template>

<style scoped>
.title {
  text-align: center;
  padding: 2rem 0;
}

.functions {
  display: flex;
  justify-content: center;
  gap: 1rem;
}
</style>
