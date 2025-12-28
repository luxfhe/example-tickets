<template>
  <div style="display: flex; flex-direction: column; align-items: center; padding: 20px">
    <div>
      <img class="logo" src="~/assets/fhenix_logo.svg" />
    </div>
    <div v-if="!isVerifier">
      <CButton v-if="!isConnected" color="warning"   @click="connect" >Connect Wallet</CButton>
      <div v-else>{{ shortAddress }}</div>
      <Buyer  v-if="isConnected" ref="buyer"  :webService="webService" :ticket="ticket" @gotKey="getKey" />          
    </div>
    <Verifier v-else :webService="webService"  />          
    
  </div>
</template>

<style scoped>
  .logo {
    height: 70px;
    margin-bottom: 20px;
  }
</style>

<script>
import { defineComponent } from '@vue/composition-api'
import Buyer from './components/Buyer.vue'
import Verifier from './components/Verifier.vue'
import { ethers } from "ethers";
import ABI from './assets/abi.json';
import { CButton } from '@coreui/vue';

import '@coreui/coreui/dist/css/coreui.min.css'



const queryString  = window.location.search;
const urlParams = new URLSearchParams(queryString);

const CHAIN_ID = 5432;
const RPC_DEFAULT_ENDPOINT = "https://fhenode.fhenix.io/new/evm";
const BLOCK_EXPLORER = "https://demoexplorer.fhenix.io";
const WEB_SERVICE = "https://ticketing-service-test.azurewebsites.net";
const CONTRACT_ADDRESS = "0x611136338d98aAa1db0E10AAEc3315699279d0dd";

var provider;
var signer;

export default defineComponent({
  components: {Buyer, Verifier, CButton},
  setup() {
    
  },
  mounted() {
    let connectedBefore = window.localStorage.getItem('ConnectedBefore');
    if (connectedBefore) {
      this.connect();
    }
  },
  data() {
    return {
      account: "",
      isVerifier: urlParams.has('verifier'),
      webService: WEB_SERVICE,
      ticket: ""
    }
  },
  computed: {
    isConnected() {
      return this.account !== "";
    },
    shortAddress() {
      if (this.isConnected) {
        return this.account.slice(0, 9) + '…' + this.account.slice(this.account.length - 6);
      }
      return "";
    },    
  },
  methods: {
    async connect() {
      provider = new ethers.BrowserProvider(window.ethereum);
      let accounts = await provider.send("eth_requestAccounts", []);
      if (accounts && accounts.length > 0) {
        this.account = accounts[0];
        try {
          await provider.send('wallet_switchEthereumChain', [{ chainId: `0x${CHAIN_ID.toString(16)}` }]);
        } catch (err) {
          await provider.send('wallet_addEthereumChain', [
            {
                chainId: `0x${CHAIN_ID.toString(16)}`,
                chainName: 'Fhenix Network New',
                rpcUrls: [RPC_DEFAULT_ENDPOINT],
                nativeCurrency: {
                  name: "FHE Token",
                  symbol: "FHE",
                  decimals: 18
                },
                blockExplorerUrls: [BLOCK_EXPLORER]
              }
            ]); 
        }

        try {
          signer = await provider.getSigner();
          window.localStorage.setItem('ConnectedBefore', "1");
        } catch (err) {
          console.log(err);
        }
        
      }
    },

    async getKey(key) {
      try {
        this.ticket = "";
        const contract = new ethers.Contract(CONTRACT_ADDRESS, ABI, signer);
        let challenge = await contract.getKeyWithChallenge(`0x${key}`);
        console.log(challenge);
        this.ticket = challenge;
      } catch (err) {
        console.log(err);
      }
      this.$refs.buyer.processingTicketDone();      
    }

  }
});
</script>
