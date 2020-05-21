<template>
    <el-button type='info' @click="LoginOut">EXIT</el-button>
</template>

<script>
export default {
  name:'',
  data(){
   return {

   }
  },
  methods: {
      LoginOut(){
          window.sessionStorage.clear();
          this.$router.push('/login')
      }
  }
}
</script>

<style lang='less' scoped>

</style>
