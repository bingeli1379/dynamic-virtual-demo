<script setup></script>

<template>
  <main>
    <h1 class="title">Dynamic Virtual Scroll</h1>

    <div class="container">
      <div class="scroll-wrap">scroll</div>
    </div>
  </main>
</template>

<style scoped>
.title {
  text-align: center;
  padding: 2rem 0;
}

.container {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1rem;
}

.scroll-wrap {
  width: 75%;
  height: 70vh;
  border: 1px solid #333333;
}
</style>
