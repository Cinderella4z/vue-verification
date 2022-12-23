<template>

  <verificationBox :success="success" :width="400" />


</template>
<script setup>
import verificationBox from './components/verificationBox/index.vue'

const success = () => {
  console.log(1);
}
</script>
<style  scoped>

</style>