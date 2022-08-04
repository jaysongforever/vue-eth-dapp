<!-- @format -->

<template>
  <div class="hello">
    <button v-if="!isConnect" @click="connect">链接web3</button>
    <button v-else @click="disConnect">断开链接</button>
    <div>
      <div>账户地址: {{ currentAccount || '-' }}</div>
      <div>账户余额: {{ currentAmount }}</div>
      <div>网络Id: {{ network.chainId }}</div>
      <div>网络名称: {{ network.name }}</div>
      <ReadContract :isConnect="isConnect" :currentAccount="currentAccount" :addressContract="addressContract" />
      <TransferERC20 :isConnect="isConnect" :currentAccount="currentAccount" :addressContract="addressContract" />
    </div>
  </div>
</template>

<script>
import { ethers } from "ethers";
import ReadContract from './ReadContract.vue'
import TransferERC20 from './TransferERC20.vue'

export default {
  name: 'HelloWorld',
  components: {
    ReadContract,
    TransferERC20
  },
  data() {
    return {
      currentAccount: '',
      currentAmount: 0,
      provider: null,
      network: {},
      isConnect: false,
      addressContract: '0x5FbDB2315678afecb367f032d93F642f64180aa3'
    }
  },
  methods: {
    async connect() {
      //client side code
      if (!window.ethereum) {
        console.log('please install MetaMask')
        return
      }
      /*
        //change from window.ethereum.enable() which is deprecated
        //see docs: https://docs.metamask.io/guide/ethereum-provider.html#legacy-methods
        window.ethereum.request({ method: 'eth_requestAccounts' })
        .then((accounts:any)=>{
          if(accounts.length>0) setCurrentAccount(accounts[0])
        })
        .catch('error',console.error)
      */

      //we can do it using ethers.js
      this.provider = new ethers.providers.Web3Provider(window.ethereum)
      try {
        const accounts = await this.provider.send('eth_requestAccounts', [])
        console.log('🚀 ~ file: HelloWorld.vue ~ line 60 ~ connect ~ accounts', accounts)
        if (accounts.length > 0) {
          this.isConnect = true
          this.setCurrentAccount(accounts[0])
          this.initData()
        }
      } catch (error) {
        this.isConnect = false
        console.log('🚀 ~ file: HelloWorld.vue ~ line 46 ~ connect ~ error', error)
      }
      // MetaMask requires requesting permission to connect users accounts
    },
    setCurrentAccount(account) {
      this.currentAccount = account
    },
    async initData() {
      if (!this.currentAccount) return
      if (!this.provider) return
      const balance = await this.provider.getBalance(this.currentAccount)
      // { BigNumber: "182826475815887608" }
      // Often you need to format the output to something more user-friendly,
      // such as in ether (instead of wei)
      this.currentAmount = ethers.utils.formatEther(balance)
      // '0.182826475815887608'
      const res = await this.provider.getNetwork()
      this.network = res
    },
    disConnect() {
      this.isConnect = false
      this.currentAccount = ''
      this.currentAmount = 0
      this.provider = null
      this.network = {}
    }
  },
}
</script>

<style scoped></style>
