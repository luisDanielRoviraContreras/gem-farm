<template>
  <button class="btn" :disabled="disabled">
    <!-- <img
      src="../assets/play.png"
      alt="right"
      :width="60"
      :height="60"
      :class="{ left: left, disabled: disabled }"
    />-->
    <div v-if="!left">
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="24"
        height="24"
        viewBox="0 0 24 24"
        style="fill: rgba(0, 0, 0, 1);transform: ;msFilter:;"
      >
        <path
          d="m11.293 17.293 1.414 1.414L19.414 12l-6.707-6.707-1.414 1.414L15.586 11H6v2h9.586z"
        />
      </svg>
    </div>
    <div v-else>
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="24"
        height="24"
        viewBox="0 0 24 24"
        style="fill: rgba(0, 0, 0, 1);transform: ;msFilter:;"
      >
        <path
          d="M12.707 17.293 8.414 13H18v-2H8.414l4.293-4.293-1.414-1.414L4.586 12l6.707 6.707z"
        />
      </svg>
    </div>
  </button>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
export default defineComponent({
  props: {
    left: Boolean,
    disabled: Boolean,
  },
  setup() {
    return {};
  },
});
</script>

<style scoped>
button,
img {
  outline: none !important;
}
.btn {
  width: 50px;
  height: 50px;
  background: #f7ff04;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 0px;
  margin: 10px;
  border-radius: 15px 0px 15px 0px;
}
.btn svg {
  width: 35px;
}
.left {
  transform: scaleX(-1);
}
.disabled {
  opacity: 0;
}
</style>
