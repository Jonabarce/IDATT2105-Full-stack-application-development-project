<script setup>
  import Header from '../components/Header.vue'
  import Footer from '../components/Footer.vue'
  import Purchase from '../components/Purchase.vue'
</script>

<template>
  <Header />
  <Purchase />
  <Footer id="footer" />
</template>

<style scoped>
    #footer{
        transform: translateY(2em)
    }
</style>