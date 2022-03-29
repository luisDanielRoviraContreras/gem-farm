<template>
  <div class="app">
    <!-- <TheNavBar /> -->
    <!-- <div class="pt-10 px-10 flex justify-center align-middle">
      <p class="text-4xl pt-3 px-1 mt-1 text-black underline">GEM</p>
      <img :width="50" :height="50" src="./assets/400x600.gif" />
      <p class="text-4xl pt-3 px-2 mt-1 text-black underline">FARM</p>
    </div>-->
    <!-- <div class="italic mt-5 text-center">by Gemworks</div> -->

    <div class="p-10">
      <router-view />
    </div>

    <div class="pt-10"></div>
    <!-- <TheCat /> -->
    <!-- <TheFooter /> -->
  </div>
</template>

<script>
import TheNavBar from '@/components/TheNavBar';
import TheFooter from '@/components/gem-farm/TheFooter';
import TheCat from '@/components/gem-farm/TheCat';
export default {
  components: { TheCat, TheFooter, TheNavBar },
};
</script>

<style>
* {
  font-family: arial, monospace;
}
input[type="radio"]:checked + span {
  @apply text-black;
}
.app {
  /* max-width: 100vw;
  max-height: 100vh;
  width: 100%;
  height: 100vh;
  /* overflow: hidden;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center; */
}
.video {
  position: absolute;
  z-index: -2;
  left: 0px;
  top: 0px;
  right: 0px;
  bottom: 0px;
  width: 100%;
  display: block;
  padding: 0px;
  margin: 0px;
  max-width: 100% !important;
  min-width: 100%;
  min-height: 100%;
  width: 100%;
}
.imgx {
  position: absolute;
  z-index: -1;
  left: 0px;
  top: -90px;
  right: 0px;
  bottom: 0px;
  width: 100%;
  display: block;
  padding: 0px;
  margin: 0px;
  max-width: 100% !important;
  min-width: 100%;
  min-height: 100%;
  width: 100%;
  opacity: 1;
}
</style>
