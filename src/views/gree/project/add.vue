<template> 
  <product-detail :is-edit='false'></product-detail>
</template>
<script>
  import ProductDetail from './components/ProjectDetail'
  export default {
    name: 'addProject',
    components: { ProductDetail }
  }
</script>
<style>
</style>
