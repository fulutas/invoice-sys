<template>
  <header class="flex">
    <router-link class="nav-link flex" :to="{ name: 'Home' }">
    <div class="branding flex">
      <img src="@/assets/file-invoice-dollar-solid.png" alt="" />
      <span class="flex">Faturalar</span>
    </div>
    </router-link>
  </header>
</template>

<script>
export default {
  name: "navigation",
};
</script>

<style lang="scss" scoped>
header {
  z-index: 99;
  flex-direction: row;
  background-color: #1e2139;
  @media (min-width: 900px) {
    min-height: 100%;
    min-width: 90px;
    flex-direction: column;
    border-radius: 0 20px 20px 0;
  }

  .branding{
      border-radius: 0 20px 20px 0;
      background-color: #7c5dfa;
      justify-content: center;
      padding: 24px;
      @media(min-width:900px){
          width: 100%;
      }

      &:hover{
        opacity: 0.9;
      }

      span{
        margin-left: 10px;
        justify-content: center;
        align-items: center;
        color: #FFF;
      }

      img {
          width: auto;
          height: 30px;
      }
  }
}
</style>
