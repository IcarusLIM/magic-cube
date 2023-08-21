<template>
  <header>
    <div class="clip-font">
      <span class="">For</span>
      <span class="font-youyuan">测试</span>
    </div>
  </header>

  <main>
    <Cube />
    <!-- <Test /> -->

    <!-- Navie UI Dialog -->
    <n-dialog-provider>
      <content />
    </n-dialog-provider>
  </main>

  <footer>
    <div class="clip-font">测试测试～</div>
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
  font-family: 'cera-round-pro-bold';
  background-clip: text;
  -webkit-background-clip: text;
  /* color: transparent; */
  background-image: url("./assets/fontbg.png");
}

.font-youyuan {
  font-family: 'Hannotate SC', 'Yuanti SC';
}
</style>
