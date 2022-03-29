<template>
  <div
    class="cardx"
    :class="{ 'card-selected': selected, info, 'notSelected': notSelected }"
    @click="toggleSelect"
    v-if="nft.externalMetadata.name.indexOf('Bizarre Platypus') != -1"
  >
    <img :src="nft.externalMetadata.image" :alt="nft.onchainMetadata.data.name" />

    <div class="info-text" v-if="info">{{ nft.externalMetadata.name }}</div>
    <!-- <p class="data">{{ nft.externalMetadata }}</p> -->
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';
export default defineComponent({
  props: {
    nft: { type: Object, required: true },
    info: Boolean,
    notSelected: Boolean,
  },
  emits: ['selected'],
  setup(props, ctx) {
    const selected = ref<boolean>(false);

    const toggleSelect = () => {
      if (props.notSelected) return
      selected.value = !selected.value;
      console.log('selected', props.nft.mint.toBase58());
      ctx.emit('selected', {
        nft: props.nft,
        selected: selected.value,
      });
    };

    return {
      selected,
      toggleSelect,
    };
  },
});
</script>

<style scoped>
.data {
  font-size: 0.6rem;
  margin-top: 150px;
}
img {
  max-width: 100%;
  max-height: 100%;
  height: auto;
  width: auto;
}

.info-text {
  position: absolute;
  bottom: 0px;
  font-size: 14px;
}
.cardx {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 300px;
  height: 300px;
  position: relative;
  margin-left: 5px;
  margin-right: 5px;
}
.cardx img {
  width: 100%;
  border-radius: 30px 0px 30px 0px;
  display: block;
}

.card:hover {
  @apply border-4 border-solid border-gray-200;
}

.card-selected {
  @apply border-4 border-solid;
  background: #000;
  border-radius: 30px 0px 30px 0px;
}
</style>
