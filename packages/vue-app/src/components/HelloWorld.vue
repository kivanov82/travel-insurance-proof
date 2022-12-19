<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <button v-if="!provider" @click="contractCall">Connect wallet</button>
    <p v-if="provider">Address: {{address}}</p>
    <p v-if="successMessage" style="color: greenyellow">{{successMessage}}</p>
    <div v-if="provider && !populated">
      <h4>Describe the products you want to record</h4>
      <input type="text" style="width: 250px" v-model="myGoodsFull[0][0]"  />
      <input type="text" style="width: 60px" v-model="myGoodsFull[0][1]" />
      <br>
      <input type="text" style="width: 250px" v-model="myGoodsFull[1][0]"  />
      <input type="text" style="width: 60px" v-model="myGoodsFull[1][1]" />
      <br>
      <input type="text" style="width: 250px" v-model="myGoodsFull[2][0]"  />
      <input type="text" style="width: 60px" v-model="myGoodsFull[2][1]" />
      <br>
      <input type="checkbox" id="checkbox" v-model="checked" />
      <label for="checkbox"> I give access to find the respective items in my bank statements and send a proof of purchase to my wallet</label>
      <br>
      <button style="" :disabled="!checked" @click="populate">Send to my wallet</button>

    </div>
    <div v-if="populated" style="border: 1px solid white; margin: auto;
  width: 25%; ">
      <p>{{goodsLabel}}</p>
      <hr>
      <div style="float: left; padding-left: 20px">{{ myGoods[0][0] }}</div>
      <div style="float: right; padding-right: 20px"> {{ myGoods[0][1] }}
        <a href="#" @click="showHistory(0)" style="padding-left: 10px"> {{ myGoods[0][2] }} </a>
      </div>
      <br>
      <div style="float: left; padding-left: 20px">{{ myGoods[1][0] }}</div>
      <div style="float: right; padding-right: 20px"> {{ myGoods[1][1] }}
        <a href="#" @click="showHistory(1)" style="padding-left: 10px"> {{ myGoods[0][2] }} </a>
      </div><br>
      <div style="float: left; padding-left: 20px">{{ myGoods[2][0] }}</div>
      <div style="float: right; padding-right: 20px"> {{ myGoods[2][1] }}
        <a href="#" @click="showHistory(2)" style="padding-left: 10px"> {{ myGoods[0][2] }} </a>
      </div><br>
      <br>
      Total: {{ myGoods[3][0] }}
      <br><br>
      <button style="" @click="shareDetails">Insure</button>
    </div>
    <div v-if="historyTrack" style="float: right; padding-right: 100px; margin-top: -300px">
      <h3>Proof of payment:</h3>
      <p>From: <b>Creditcard</b> 5423********3476 </p>
      <p>Amazon NL</p>
      <p>Date: {{ historyTrack.date }}</p>
      <p>Amount: <b>{{ historyTrack.amount }}</b></p>
    </div>
  </div>
</template>

<script>
  import { Contract } from "@ethersproject/contracts";
  import { Web3Provider } from "@ethersproject/providers";
  import { addresses, abis } from "@my-app/contracts";
  import GET_TRANSFERS from "../graphql/subgraph";

  export default {
    methods: {
      contractCall: async function () {
        // Should replace with the end-user wallet, e.g. Metamask
        await window.ethereum.enable();
        const defaultProvider = new Web3Provider(window.ethereum);
        // Create an instance of an ethers.js Contract
        // Read more about ethers.js on https://docs.ethers.io/v5/api/contract/contract/
        const ceaErc20 = new Contract(addresses.ceaErc20, abis.erc20, defaultProvider);
        // A pre-defined address that owns some CEAERC20 tokens
        const tokenBalance = await ceaErc20.balanceOf("0x3f8CB69d9c0ED01923F11c829BaE4D9a4CB6c82C");
        console.log({ tokenBalance: tokenBalance.toString() });
        this.provider = defaultProvider;
        this.signer = await this.provider.getSigner();
        this.address = await this.signer.getAddress();
      },
      shareDetails: async function () {
        this.signer.signMessage('Hereby I authorize Generali to see the details about my goods for the next 24 hours starting now (' + new Date().toString() + ' )').then(() => {
          this.myGoods = this.myGoodsFull;
          this.goodsLabel = 'My insured goods';
          this.successMessage = 'Success! Your goods are insured with Generali, and an NFT with proof on insurance was sent to your wallet!';
        });
      },
      showHistory: function(item) {
        this.signer.signMessage('I give Generali access to find the respective items in my bank statements for my item number ' + item + ' from my goods list').then(() => {
          this.historyTrack = {
            amount: this.myGoods[item][1],
            date: this.myGoods[item][3],
          }
          this.successMessage = 'Your claim was submitted and the proof of payment was shared with Generali'
        });
      },
      populate: function() {
        /*this.signer.sendTransaction({
          to: this.address,
          value: "1"
        }).then(() => {
          this.populated = true
          this.successMessage = 'Success! Your items are found. Would you like to insure them?'
        });*/
        this.populated = true
        this.successMessage = 'Success! Your items are found and privately stored in your NFT. Would you like to insure them?'
      }
    },
    name: "HelloWorld",
    props: {
      msg: String,
    },
    data () {
      return {
        provider: null,
        address: null,
        signer: null,
        checked: false,
        populated: false,
        successMessage: '',
        goodsLabel: 'My goods',
        myGoods: [
            ['******', '***', ''],
          ['******', '***', ''],
          ['******', '***', ''],
          ['*** €']
        ],
        myGoodsFull: [
          ['MacBook Pro 16-inch, 2021', '2340 €', '>>>Claim', '05/12/2022'],
          ['Sony ZV-1F Camera', '649 €', '>>>Claim', '02/06/2022'],
          ['GoPro HERO 11 Black', '449 €', '>>>Claim', '05/01/2021'],
          ['3408 €'],
        ],
        historyTrack: null,
      }
    },
    async mounted() {
      try {
        const { data, loading, stale } = await this.$apollo.query({
          query: GET_TRANSFERS,
        });
        if (!loading && !stale && data && data.transfers) {
          console.log({ transfers: data.transfers });
        }
      } catch (error) {
        console.error("Error while pulling data from the subgraph:", error);
      }
    },
  };
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  h1 {
    margin-top: 24px;
  }

  h3 {
    margin: 40px 0 0;
  }

  ul {
    list-style-type: none;
    padding: 0;
  }

  li {
    display: inline-block;
    margin: 0 10px;
  }

  a {
    color: #42b983;
  }
</style>
