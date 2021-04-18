<template>
  <Calculator />
</template>

<script>
import Calculator from './components/Calculator.vue'

export default {
  name: 'App',
  components: {
    Calculator
  }
}
</script>

<style>
/* Reset */
*,*::after,*::before{
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}
</style>
