<template>
  <header>
    <div class="clip-font">
      <span class="font-en">For </span>
      <span class="font-cn"> 可乐盖儿</span>
    </div>
  </header>

  <main>
    <Cube />
    <!-- <Test /> -->
  </main>

  <footer>
    <div style="width: 100%;display: flex;">
      <div class="clip-font" style="margin-left: auto;">
        <span class="font-cn">七夕快乐呀～</span>
      </div>
    </div>
  </footer>
</template>

<script setup lang="ts">
import Cube from './components/Cube.vue'
import Test from './components/Test.vue'
</script>

<style scoped>
header {
  line-height: 1.5;
}

.clip-font {
  background-clip: text;
  -webkit-background-clip: text;
  color: transparent;
  background-image: url("./assets/bg.png");
}

.font-en {
  font-family: 'cera-round-pro-bold';
  font-size: 50px;
}

.font-cn {
  font-family: 'Hannotate SC', 'Yuanti SC';
  font-size: 30px;
}
</style>
