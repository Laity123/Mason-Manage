<template>
  <div class="welcome_page">
    <div class="welcome_img">
      <img src="../assets/logo.png" alt />
    </div>
    <h1 class="welcome_des">Welcome.</h1>
  </div>
</template>

<script>
export default {
  name: "",
  data() {
    return {};
  }
};
</script>

<style lang='less' scoped>
.welcome_page {
  box-sizing: border-box;
  padding: 150px;
  background-color: #2f3542;
}
.welcome_img {
  width: 200px;
  height: 200px;
  border-radius: 50%;
  border: 4px solid black;
  margin: auto;
  > img {
    width: 100%;
    height: 100%;
    border-radius: 50%;
  }
}
.welcome_des {
  font-size: 40px;
  font-weight: 700;
  text-align: center;
}
</style>
