<template>
  <div class="main">
    <!-- <div class="title">Buyer</div> -->
    <div>
      <CButton color="success" :disabled="purchasing"  @click="purchaseTicket">{{ purchaseButtonTitle }}</CButton>
    </div>
    <div v-if="hasTicket && !purchasing" style="position: relative; margin-top: 20px">
      <img style="height: 399px" src="~/assets/ticket.svg" />
      <div class="qr-container">
        <qrcode-vue render-as="svg" background="none" style="width: 100%; height: 100%;" :value="ticket" :size="300" level="H" />
      </div>
      <CButton style="position: absolute; bottom: 92px; left: 80px" color="secondary" :disabled="purchasing" size="sm" @click="refreshTicket">Refresh</CButton>
    </div>
    <div v-if="purchasing" style="margin-top: 20px"> 
      <CSpinner style="width: 50px; height: 50px"  color="warning"/>
    </div>
  </div>
</template>

<script>
import QrcodeVue from 'qrcode.vue'
import { CButton, CSpinner } from '@coreui/vue';

export default {
  name: 'Buyer',
  components: { QrcodeVue, CButton, CSpinner },
  props: {
    webService: String,
    ticket: String
  },
  created() {
  },
  mounted() {
    var self = this;
    setTimeout(() => {
      this.ticketData = window.localStorage.getItem('MyTicket');
      if (this.ticketData) {
        this.purchasing = true;
        this.$emit('gotKey', this.ticketData);
      }
    }, 1000);
  },
  data() {
    return {
      purchasing: false,
      ticketData: ""
    }
  },
  computed: {
    hasTicket() {
      return (this.ticket !== "" && this.ticket !== undefined);
    },
    purchaseButtonTitle() {
      return this.purchasing ? "Please wait..." : ( !this.hasTicket  ? "Purchase a ticket" : "Purchase a new ticket" );
    }
  },
  methods: {
    processingTicketDone() {
      this.purchasing = false;
    },
    refreshTicket() {
      this.$emit('gotKey', this.ticketData);
    },
    async purchaseTicket() {
      try {
        this.purchasing = true;
        const encodedData = await this.$axios.get(`${this.webService}/create`);
        if (encodedData.status !== 200) {
          console.log(`Failed to encrypt number from service: ${encodedData.status} ${encodedData.statusText}`);
          return;
        }
        this.ticketData = encodedData.data;
        window.localStorage.setItem('MyTicket', this.ticketData);
        this.$emit('gotKey', this.ticketData);
      } catch (err) {
        console.log(err);
      }    
    }

  }
}
</script>

<style>
  :root {
    --qr-size: 160px;  
  }
</style>

<style scoped>

  .main {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .title {
    font-size: 20px;
    font-weight: bold;
    margin-bottom: 20px;
  }

  .qr-container {
    position: absolute;
    top: 50px;
    left: 35px;
    width: var(--qr-size);
    height: var(--qr-size);
  }
</style>