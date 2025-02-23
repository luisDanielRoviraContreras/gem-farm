<template>
  <!--control buttons-->
  <div class="content-popup">
    <div class="con-popup">
      <slot></slot>
      <button class="close" @click="handleClose">
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
          <path
            d="m16.192 6.344-4.243 4.242-4.242-4.242-1.414 1.414L10.535 12l-4.242 4.242 1.414 1.414 4.242-4.242 4.243 4.242 1.414-1.414L13.364 12l4.242-4.242z"
          />
        </svg>
      </button>
      <div class="popup">
        <div v-if="desiredVaultNFTs.length > 0"></div>

        <button @click="refresh">Refresh {{ isMove }}</button>
      </div>

      <!--wallet + vault view-->
      <div class="flex items-stretch">
        <!--left-->
        <NFTGrid
          title="Selected your NFT"
          class="flex-1"
          :nfts="desiredWalletNFTs"
          @selected="handleWalletSelected"
          move
        >
          <button
            v-if="selectedNFTs.length > 0 && farmerState === 'staked'"
            @click="$emit('add-gems')"
          >Add Bizarre Platypus (resets staking)</button>
          <!-- <button @click="$emit('add-gems')">Add Bizarre Platypus (resets staking)</button> -->
        </NFTGrid>

        <!--mid-->
        <div class="m-2 flex flex-col">
          <ArrowButton :disabled="vaultLocked" class="my-2" @click="moveNFTsFE(false)" />
          <ArrowButton :disabled="vaultLocked" class="my-2" :left="true" @click="moveNFTsFE(true)" />
        </div>

        <!--right-->
        <NFTGrid
          title="NFTS IN THE TRUNK"
          class="flex-1"
          :nfts="desiredVaultNFTs"
          :move="isMove"
          :farmerState="farmerState"
          @selected="handleVaultSelected"
        >
          <!-- <div v-if="vaultLocked" class="locked">
            <p class="mt-10">This vault is locked!</p>
          </div>-->
          <button
            v-if="
              (toWalletNFTs && toWalletNFTs.length) ||
              (toVaultNFTs && toVaultNFTs.length)
            "
            @click="moveNFTsOnChain"
          >Move Bizarre Platypus!</button>
          <button v-if="farmerState == 'unstaked'" @click="$emit('start')">Start Staking</button>
          <button v-if="farmerState == 'pendingCooldown'" @click="$emit('end-staking')">End cooldown</button>
          <p>{{ farmerState }}</p>
        </NFTGrid>

        <NFTGrid
          title="NFTS IN STAKING"
          class="flex-1"
          :nfts="desiredVaultNFTs"
          @selected="handleVaultSelected"
          :farmerState="farmerState"
          :staking="true"
          :notSelected="true"
        >
          <button v-if="farmerState == 'staked'" @click="$emit('end-staking')">Unstaked</button>
        </NFTGrid>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref, watch } from 'vue';
import NFTGrid from '@/components/gem-bank/NFTGrid.vue';
import ArrowButton from '@/components/ArrowButton.vue';
import useWallet from '@/composables/wallet';
import useCluster from '@/composables/cluster';
import {
  getNFTMetadataForMany,
  getNFTsByOwner,
  INFT,
} from '@/common/web3/NFTget';
import { initGemBank } from '@/common/gem-bank';
import { PublicKey } from '@solana/web3.js';
import { getListDiffBasedOnMints, removeManyFromList } from '@/common/util';
import { BN } from '@project-serum/anchor';
import NFTStaked from '@/components/gem-bank/NFTStaked.vue';
import RefreshFarmerVue from '../gem-farm/RefreshFarmer.vue';

export default defineComponent({
  components: { ArrowButton, NFTGrid, NFTStaked },
  props: {
    farmerState: String,
    vault: String,
    selectedNFTs: Array,
  },
  emits: ['selected-wallet-nft', 'moveimage', 'close', 'end-staking', 'add-gems', 'start'],
  setup(props, ctx) {
    const { wallet, getWallet } = useWallet();
    const { cluster, getConnection } = useCluster();

    // --------------------------------------- state

    //current walet/vault state
    const currentWalletNFTs = ref<INFT[]>([]);
    const currentVaultNFTs = ref<INFT[]>([]);
    //selected but not yet moved over in FE
    const selectedWalletNFTs = ref<INFT[]>([]);
    const selectedVaultNFTs = ref<INFT[]>([]);
    //moved over in FE but not yet onchain
    const desiredWalletNFTs = ref<INFT[]>([]);
    const desiredVaultNFTs = ref<INFT[]>([]);
    //moved over onchain
    const toWalletNFTs = ref<INFT[]>([]);
    const toVaultNFTs = ref<INFT[]>([]);

    const isMove = ref(false)

    const refresh = async () => {
      console.log('paso por refresh')
      await Promise.all([populateWalletNFTs(), populateVaultNFTs()]);
    }

    // --------------------------------------- populate initial nfts

    const populateWalletNFTs = async () => {
      // zero out to begin with
      currentWalletNFTs.value = [];
      selectedWalletNFTs.value = [];
      desiredWalletNFTs.value = [];

      if (getWallet()) {
        currentWalletNFTs.value = await getNFTsByOwner(
          getWallet()!.publicKey!,
          getConnection()
        );
        // console.log([...currentWalletNFTs.value])
        // BfD5aPbyzCpmvcaCDapqPGrH39Vpt7ZDJnhpQk2WBspT
        desiredWalletNFTs.value = [...currentWalletNFTs.value];
      }
    };

    const populateVaultNFTs = async () => {
      // zero out to begin with
      currentVaultNFTs.value = [];
      selectedVaultNFTs.value = [];
      desiredVaultNFTs.value = [];

      const foundGDRs = await gb.fetchAllGdrPDAs(vault.value);
      if (foundGDRs && foundGDRs.length) {
        gdrs.value = foundGDRs;
        console.log(`found a total of ${foundGDRs.length} gdrs`);

        const mints = foundGDRs.map((gdr: any) => {
          console.log('gdr')
          console.log(gdr)
          console.log('gdr')
          return { mint: gdr.account.gemMint };
        });
        currentVaultNFTs.value = await getNFTMetadataForMany(
          mints,
          getConnection()
        );
        desiredVaultNFTs.value = [...currentVaultNFTs.value];
        console.log(
          `populated a total of ${currentVaultNFTs.value.length} vault NFTs`
        );
      }
    };

    const updateVaultState = async () => {
      vaultAcc.value = await gb.fetchVaultAcc(vault.value);
      bank.value = vaultAcc.value.bank;
      vaultLocked.value = vaultAcc.value.locked;
    };

    watch([wallet, cluster], async () => {
      gb = await initGemBank(getConnection(), getWallet()!);

      //populate wallet + vault nfts
      await Promise.all([populateWalletNFTs(), populateVaultNFTs()]);
    });

    onMounted(async () => {
      gb = await initGemBank(getConnection(), getWallet()!);

      //prep vault + bank variables
      vault.value = new PublicKey(props.vault!);
      await updateVaultState();

      //populate wallet + vault nfts
      await Promise.all([populateWalletNFTs(), populateVaultNFTs()]);
    });

    // --------------------------------------- moving nfts

    const handleWalletSelected = (e: any) => {
      if (e.selected) {
        selectedWalletNFTs.value.push(e.nft);
      } else {
        const index = selectedWalletNFTs.value.indexOf(e.nft);
        selectedWalletNFTs.value.splice(index, 1);
      }
      ctx.emit('selected-wallet-nft', selectedWalletNFTs.value);
    };

    const handleVaultSelected = (e: any) => {
      if (e.selected) {
        selectedVaultNFTs.value.push(e.nft);
      } else {
        const index = selectedVaultNFTs.value.indexOf(e.nft);
        selectedVaultNFTs.value.splice(index, 1);
      }
    };

    const moveNFTsFE = (moveLeft: boolean) => {

      if (moveLeft) {
        //push selected vault nfts into desired wallet
        desiredWalletNFTs.value.push(...selectedVaultNFTs.value);
        //remove selected vault nfts from desired vault
        removeManyFromList(selectedVaultNFTs.value, desiredVaultNFTs.value);
        //empty selection list
        selectedVaultNFTs.value = [];
      } else {
        //push selected wallet nfts into desired vault
        desiredVaultNFTs.value.push(...selectedWalletNFTs.value);
        //remove selected wallet nfts from desired wallet
        removeManyFromList(selectedWalletNFTs.value, desiredWalletNFTs.value);
        //empty selected walelt
        selectedWalletNFTs.value = [];
      }
    };

    //todo jam into single tx
    const moveNFTsOnChain = async () => {
      for (const nft of toVaultNFTs.value) {
        console.log(nft);
        const creator = new PublicKey(
          //todo currently simply taking the 1st creator
          (nft.onchainMetadata as any).data.creators[0].address
        );
        console.log('creator is', creator.toBase58());
        await depositGem(nft.mint, creator, nft.pubkey!);
      }
      for (const nft of toWalletNFTs.value) {
        await withdrawGem(nft.mint);
      }
      await Promise.all([populateWalletNFTs(), populateVaultNFTs()]);
    };

    //to vault = vault desired - vault current
    watch(
      desiredVaultNFTs,
      () => {
        toVaultNFTs.value = getListDiffBasedOnMints(
          desiredVaultNFTs.value,
          currentVaultNFTs.value
        );
        console.log('to vault nfts are', toVaultNFTs.value);
      },
      { deep: true }
    );

    //to wallet = wallet desired - wallet current
    watch(
      desiredWalletNFTs,
      () => {
        toWalletNFTs.value = getListDiffBasedOnMints(
          desiredWalletNFTs.value,
          currentWalletNFTs.value
        );
        console.log('to wallet nfts are', toWalletNFTs.value);
      },
      { deep: true }
    );

    // --------------------------------------- gem bank

    let gb: any;
    const bank = ref<PublicKey>();
    const vault = ref<PublicKey>();
    const vaultAcc = ref<any>();
    const gdrs = ref<PublicKey[]>([]);
    const vaultLocked = ref<boolean>(false);

    const depositGem = async (
      mint: PublicKey,
      creator: PublicKey,
      source: PublicKey
    ) => {
      const { txSig } = await gb.depositGemWallet(
        bank.value,
        vault.value,
        new BN(1),
        mint,
        source,
        creator
      );
      console.log('deposit done', txSig);
      ctx.emit('moveimage')
    };

    const withdrawGem = async (mint: PublicKey) => {
      const { txSig } = await gb.withdrawGemWallet(
        bank.value,
        vault.value,
        new BN(1),
        mint
      );
      console.log('withdrawal done', txSig);
    };

    const handleClose = () => {
      console.log('click')
      ctx.emit('close');
    }

    // --------------------------------------- return

    return {
      wallet,
      desiredWalletNFTs,
      desiredVaultNFTs,
      toVaultNFTs,
      toWalletNFTs,
      handleWalletSelected,
      handleVaultSelected,
      moveNFTsFE,
      moveNFTsOnChain,
      bank,
      // eslint-disable-next-line vue/no-dupe-keys
      vault,
      vaultLocked,
      handleClose,
      refresh,
    };
  },
});
</script>

<style>
.close {
  position: absolute;
  right: 0px;
  top: 0px;
  width: 50px;
  height: 50px;
  border: 0px;
  background: #000;
  color: #fff;
  border-radius: 10px;
  border: 2px solid #fff;
  box-shadow: 0px 5px 20px 0px rgba(255, 255, 255, 0.15);
}
.close svg {
  width: 30px;
  fill: #fff;
}
.locked {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  opacity: 0.7;
  z-index: 10;
  background: rgba(0, 0, 0, 0.7);
  color: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
}
/* .con-popup {
  position: fixed;
  background: #000;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
} */
.content-popup {
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.5);
  position: fixed;
  z-index: 1000;
}
.con-popup {
  position: fixed;
  left: 0px;
  top: 0px;
  backdrop-filter: saturate(180%) blur(10px);
  background: rgba(0, 0, 0, 0.55);
  border-radius: 0px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  color: #fff;
  width: 100%;
  height: 100%;
  background-image: url("~@/assets/bg-popup.png");
}
.con-popup .gridx .con-nfts {
  padding-top: 8px;
  height: 600px !important;
  background: #000;
}
.con-popup .gridx .titlex {
  margin-bottom: 0px;
}
</style>
