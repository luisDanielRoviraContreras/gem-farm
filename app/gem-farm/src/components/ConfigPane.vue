<template>
  <div class="menu-config">
    <!-- <div class="nes-select is-dark flex-1">
      <select required id="cluster" v-model="chosenCluster">
        <option :value="Cluster.Mainnet">Mainnet</option>
        <option :value="Cluster.Devnet">Devnet</option>
        <option :value="Cluster.Testnet">Testnet</option>
        <option :value="Cluster.Localnet">Localnet</option>
      </select>
    </div>-->
    <div class="selectx">
      <select required id="wallet" v-model="chosenWallet">
        <option class="text-gray-500" :value="null">Select Wallet</option>
        <option :value="WalletName.Phantom">Phantom</option>
        <option :value="WalletName.Sollet">Sollet</option>
        <option :value="WalletName.SolletExtension">Sollet Extension</option>
        <option :value="WalletName.Solflare">Solflare</option>
        <option :value="WalletName.SolflareWeb">Solflare Web</option>
      </select>
    </div>
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, onMounted, withCtx } from 'vue';
import { WalletName } from '@solana/wallet-adapter-wallets';
import useCluster, { Cluster } from '@/composables/cluster';
import useWallet from '@/composables/wallet';

export default defineComponent({
  emits: ['selected'],
  setup(props, ctx) {
    // cluster
    const { cluster, setCluster, getClusterURL } = useCluster();
    const chosenCluster = computed({
      get() {
        return cluster.value;
      },
      set(newVal: Cluster) {
        setCluster(newVal);
      },
    });

    // wallet
    const { getWalletName, setWallet } = useWallet();
    const chosenWallet = computed({
      get() {
        return getWalletName();
      },
      set(newVal: WalletName | null) {
        setWallet(newVal, getClusterURL());
        ctx.emit('selected', newVal);
        console.log('Selected');
      },
    });

    onMounted(() => {
      setCluster(Cluster.Mainnet)
      // setCluster(Cluster.Devnet)
    })

    return {
      Cluster,
      chosenCluster,
      WalletName,
      chosenWallet,
    };
  },
});
</script>

<style>
.menu-config {
  position: relative;
}
.selectx {
  padding-right: 10px;
  background: #ff00be;
  border-radius: 20px 0px 20px 0px;
}
.selectx select {
  padding: 15px 20px;
  background: #ff00be;
  border: 0px;
  color: #fff;
  padding-right: 30px;
  border-radius: inherit;
  font-weight: bold;
  min-width: 220px;
}
</style>
