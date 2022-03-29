<template>
  <div class="gridx">
    <h3 class="titlex">{{ title }}</h3>
    <!-- <p class="title"></p> -->
    <div :class="[notSelected]" class="flex flex-wrap con-nfts">
      <NFTCard
        v-for="nft in nfts"
        :key="nft"
        :nft="nft"
        :info="info"
        :notSelected="notSelected"
        @selected="handleSelected"
        v-show="staking ? farmerState == 'staked' : farmerState == 'pendingCooldown' || farmerState == 'unstaked' ? true : move"
      />
    </div>
    <slot></slot>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import NFTCard from '@/components/gem-bank/NFTCard.vue';

export default defineComponent({
  components: { NFTCard },
  emits: ['selected'],
  props: {
    title: String,
    nfts: Array,
    info: Boolean,
    farmerState: String,
    staking: Boolean,
    move: Boolean,
    notSelected: Boolean
  },
  setup(props, ctx) {
    const handleSelected = (e: any) => {
      if (props.notSelected) return
      console.log('paso de clikc selected')
      ctx.emit('selected', e);
    };
    return {
      handleSelected,
    };
  },
});
</script>

<style scoped>
.notSelected {
  pointer-events: none;
}
.gridx {
  border: 2px solid #fff;
  border-radius: 0px;
  position: relative;
  background: #000;
}
.titlex {
  text-transform: uppercase;
  text-align: center;
  width: 100%;
  background: #000;
  padding: 7px;
}
</style>
