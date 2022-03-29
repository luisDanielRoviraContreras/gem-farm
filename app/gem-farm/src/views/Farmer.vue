<template>
  <div class="con-farm">
    <img v-if="farmerState !== 'staked'" class="imgx" src="@/assets/bg.png" alt />
    <!-- <video class="video" width="100%" autoplay muted loop>
      <source src="@/assets/video.mp4" type="video/mp4" />
    </video>-->

    <!-- <div v-if="!wallet" class="text-center">Pls connect (burner) wallet</div>
    <div v-else>-->
    <!--farm address-->
    <!-- <div class="nes-container with-title mb-10">
        <p class="title">Connect to a Farm</p>
        <div class="nes-field mb-5">
          <label for="farm">Farm address:</label>
          <input id="farm" class="nes-input" v-model="farm" />
        </div>
    </div>-->

    <div class="con-data" v-if="farmerAcc">
      <NFTStaked
        v-if="farmerState === 'staked'"
        :key="farmerAcc"
        class="mb-10"
        @click="openPopup = true"
        :vault="farmerAcc.vault.toBase58()"
        @selected-wallet-nft="handleNewSelectedNFT"
      >
        <button
          v-if="farmerState === 'staked'"
          class="btn-unstake"
          @click="openPopup = true"
        >UNSTAKE</button>
      </NFTStaked>
      <div v-else></div>

      <div v-if="farmerState !== 'staked'" @click="openPopup = true" class="btn-add">+</div>

      <div class="con-info">
        <div class="titlex">Earnings</div>
        <div class="valuex">{{ getDecimals(availableA) }} $CYRUS</div>
        <!-- <div class="valuex">{{ availableB }} $CYRUS 2</div> -->

        <div class="titlex">Bizarre Platypus Staked</div>
        <div class="valuex">{{ farmerAcc.gemsStaked }}</div>

        <div class="titlex">Unlock on</div>
        <div class="valuex">{{ parseDate(farmerAcc?.minStakingEndsTs) }}</div>
        <!-- <button @click="handleRefreshFarmer">Recargar</button> -->
        <button class="btn-unstake" @click="claim">Claim $CYRUS</button>
        <FarmerDisplay
          :key="farmerAcc"
          :farm="farm"
          :farmAcc="farmAcc"
          :farmer="farmer"
          :farmerAcc="farmerAcc"
          class="mb-10"
          @refresh-farmer="handleRefreshFarmer"
        />
      </div>

      <Vault
        v-if="openPopup"
        :key="farmerAcc"
        class="vaulx"
        :farmerState="farmerState"
        :vault="farmerAcc.vault.toBase58()"
        @selected-wallet-nft="handleNewSelectedNFT"
        @moveimage="beginStaking"
        @close="openPopup = false"
        @end-staking="endStaking"
        @add-gems="addGems"
        :selectedNFTs="selectedNFTs"
        @start="start"
      >
        <h1>{{ farmerState }}</h1>
        <!-- <Button></Button> -->
        <!-- <button
          v-if="farmerState === 'staked' && selectedNFTs.length > 0"
          class="btn-unstake"
          @click="addGems"
        >Add Bizarre Platypus (resets staking)</button>-->
        <!-- <button
          v-if="farmerState === 'unstaked'"
          class="startx btn-unstake"
          @click="beginStaking"
        >Start Staking</button>-->
        <!-- <button v-if="farmerState === 'staked'" class="btn-unstake" @click="endStaking">UNSTAKE</button> -->
        <!-- <button
          v-if="farmerState === 'pendingCooldown'"
          class="btn-unstake"
          @click="endStaking"
        >End cooldown</button>-->
        <!-- <button class="nes-btn is-warning" @click="claim">Claim {{ availableA }} $CYRUS</button> -->
      </Vault>
    </div>
    <div class="con-btn-start-farm" v-else>
      <h1>STAKING</h1>
      <div v-if="!notStakeAccount" class="w-full text-center">
        <ConfigPane @selected="handleSelectedWallet" />
      </div>
      <div v-else class="w-full text-center">
        <h3>
          You need a staking
          <br />account to get started!
        </h3>
        <button class="btn-unstake blackx create-btn" @click="initFarmer">Init Staking</button>
      </div>
    </div>
  </div>
  <!-- </div> -->
</template>

<script lang="ts">
import { defineComponent, nextTick, onMounted, ref, watch } from 'vue';
import useWallet from '@/composables/wallet';
import useCluster from '@/composables/cluster';
import { initGemFarm } from '@/common/gem-farm';
import { PublicKey } from '@solana/web3.js';
import ConfigPane from '@/components/ConfigPane.vue';
import FarmerDisplay from '@/components/gem-farm/FarmerDisplay.vue';
import Vault from '@/components/gem-bank/Vault.vue';
import { INFT } from '@/common/web3/NFTget';
import { findFarmerPDA, stringifyPKsAndBNs } from '@gemworks/gem-farm-ts';
import { parseDate } from '@/common/util';
import NFTStaked from '@/components/gem-bank/NFTStaked.vue';
import Button from '@/components/Button.vue'

export default defineComponent({
  components: { Vault, FarmerDisplay, ConfigPane, NFTStaked, Button },
  setup() {
    const { wallet, getWallet } = useWallet();
    const { cluster, getConnection } = useCluster();
    const openPopup = ref(false);
    const notStakeAccount = ref(false);

    let gf: any;
    watch([wallet, cluster], async () => {
      await freshStart();
    });

    //needed in case we switch in from another window
    onMounted(async () => {
      await freshStart();
    });

    // --------------------------------------- farmer details
    const farm = ref<string>('E7Vtt9aTKB8QQiod7563aXQXRvfMLdsLaxdaj9Q7RD72');
    const farmAcc = ref<any>();

    const farmerIdentity = ref<string>();
    const farmerAcc = ref<any>();
    const farmerState = ref<string>();
    const availableA = ref<string>();
    const availableB = ref<string>();

    //auto loading for when farm changes
    watch(farm, async () => {
      await freshStart();
    });

    const updateAvailableRewards = async () => {
      availableA.value = farmerAcc.value.rewardA.accruedReward
        .sub(farmerAcc.value.rewardA.paidOutReward)
        .toString();
      availableB.value = farmerAcc.value.rewardB.accruedReward
        .sub(farmerAcc.value.rewardB.paidOutReward)
        .toString();
    };

    const fetchFarn = async () => {
      farmAcc.value = await gf.fetchFarmAcc(new PublicKey(farm.value!));
      console.log(
        `farm found at ${farm.value}:`,
        stringifyPKsAndBNs(farmAcc.value)
      );
    };

    const fetchFarmer = async () => {
      console.log('pasooooo por aqui el cambiar el valor de farmer state <=================')
      const [farmerPDA] = await findFarmerPDA(
        new PublicKey(farm.value!),
        getWallet()!.publicKey!
      );
      farmerIdentity.value = await getWallet()!.publicKey?.toBase58();
      farmerAcc.value = await gf.fetchFarmerAcc(farmerPDA);
      farmerState.value = await gf.parseFarmerState(farmerAcc.value);
      await updateAvailableRewards();
      console.log(
        `farmer found at ${farmerIdentity.value}:`,
        stringifyPKsAndBNs(farmerAcc.value)
      );
    };

    const freshStart = async () => {
      if (getWallet() && getConnection()) {
        gf = await initGemFarm(getConnection(), getWallet()!);
        farmerIdentity.value = getWallet()!.publicKey?.toBase58();

        //reset stuff
        farmAcc.value = undefined;
        farmerAcc.value = undefined;
        farmerState.value = undefined;
        availableA.value = undefined;
        availableB.value = undefined;

        try {
          await fetchFarn();
          await fetchFarmer();
        } catch (e) {
          notStakeAccount.value = true
          console.log(`farm with PK ${farm.value} not found :(`);
        }
      }
    };

    const initFarmer = async () => {
      await gf.initFarmerWallet(new PublicKey(farm.value!));
      await fetchFarmer();
    };

    const start = async () => {
      await gf.stakeWallet(new PublicKey(farm.value!));
      await fetchFarmer();
      selectedNFTs.value = [];
    }

    // --------------------------------------- staking
    const beginStaking = async () => {
      // await gf.stakeWallet(new PublicKey(farm.value!));
      await fetchFarmer();
      selectedNFTs.value = [];
    };

    const endStaking = async () => {
      await gf.unstakeWallet(new PublicKey(farm.value!));
      await fetchFarmer();
      selectedNFTs.value = [];
    };

    const claim = async () => {
      await gf.claimWallet(
        new PublicKey(farm.value!),
        new PublicKey(farmAcc.value.rewardA.rewardMint!),
        new PublicKey(farmAcc.value.rewardB.rewardMint!)
      );
      await fetchFarmer();
    };

    const handleRefreshFarmer = async () => {
      await fetchFarmer();
    };

    const handleSelectedWallet = () => {
      console.log('change wallet selected')
    }

    // --------------------------------------- adding extra gem
    const selectedNFTs = ref<INFT[]>([]);

    const handleNewSelectedNFT = (newSelectedNFTs: INFT[]) => {
      console.log(`selected ${newSelectedNFTs.length} NFTs`);
      selectedNFTs.value = newSelectedNFTs;
    };

    const addSingleGem = async (
      gemMint: PublicKey,
      gemSource: PublicKey,
      creator: PublicKey
    ) => {
      await gf.flashDepositWallet(
        new PublicKey(farm.value!),
        '1',
        gemMint,
        gemSource,
        creator
      );
      await fetchFarmer();
    };

    const addGems = async () => {
      await Promise.all(
        selectedNFTs.value.map((nft) => {
          const creator = new PublicKey(
            //todo currently simply taking the 1st creator
            (nft.onchainMetadata as any).data.creators[0].address
          );
          console.log('creator is', creator.toBase58());

          addSingleGem(nft.mint, nft.pubkey!, creator);
        })
      );
      console.log(
        `added another ${selectedNFTs.value.length} gems into staking vault`
      );
    };

    const getDecimals = (val: any) => {
      // const CFORMAT_BTC = Intl.NumberFormat('de-DE', { style: 'currency', currency: 'XBT' })
      console.log(val)
      const newVal: number = Math.floor(val / 1000000000)
      console.log(newVal)
      return newVal;
    }

    return {
      wallet,
      farm,
      notStakeAccount,
      farmAcc,
      farmer: farmerIdentity,
      farmerAcc,
      farmerState,
      availableA,
      availableB,
      initFarmer,
      beginStaking,
      endStaking,
      claim,
      handleRefreshFarmer,
      selectedNFTs,
      handleNewSelectedNFT,
      addGems,
      parseDate,
      NFTStaked,
      openPopup,
      getDecimals,
      handleSelectedWallet,
      start
    };
  },
});
</script>
<style>
.imgx {
  z-index: 0;
}
</style>
<style scoped>
.create-btn {
  background: #feff02 !important;
  color: #000 !important;
}
.con-btn-start-farm {
  position: absolute;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
}

h1 {
  font-size: 6rem;
}
.con-farm {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100vw;
  height: 100vh;
  top: 0px;
  left: 0px;
  overflow: hidden;
  position: relative;
  background: #000;
  color: #fff;
}

h3 {
  text-align: center;
}
.con-data {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100vw;
  height: 100vh;
  top: 0px;
  left: 0px;
}
.con-info {
  background: rgba(255, 255, 255, 0.1);
  border: 2px solid #fff;
  color: #fff;
  font-family: Arial, Helvetica, sans-serif !important;
  border-radius: 20px 0px 20px 0px;
  margin-right: 40px;
  backdrop-filter: saturate(180%) blur(20px);
  padding: 20px;
  min-width: 350px;
}
.titlex {
  font-size: 14px;
  padding: 10px 20px;
  font-weight: bold;
}
.valuex {
  font-size: 28px;
  padding: 0px 20px;
  font-weight: bold;
  margin-bottom: 15px;
}
.btn-add {
  position: absolute;
  left: 40px;
  width: 270px;
  height: 270px;
  backdrop-filter: saturate(180%) blur(10px);
  background: rgba(0, 0, 0, 0.55);
  border-radius: 0px 30px 0px 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 7rem;
  color: #fff;
  border: 2px solid #fff;
}
.btn-add::after {
  content: "";
  left: -10px;
  top: -10px;
  width: calc(100% + 20px);
  height: calc(100% + 20px);
  border: 2px solid #fff;
  position: absolute;
  border-radius: inherit;
}
</style>
