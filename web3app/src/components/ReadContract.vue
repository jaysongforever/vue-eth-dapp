<template>
  <div>
    <div>用户地址: {{ currentAccount }}</div>
    <div>合约地址: {{ addressContract }}</div>
    <div>总供应量: {{ totalSupply }}</div>
    <div>符号: {{ symbol }}</div>
    <div>账户余额: {{ balance }} {{ symbol }}</div>
  </div>
</template>

<script>
import { ERC20ABI as abi } from '../abi/ERC20ABI'
import { ethers } from 'ethers'

export default {
  name: 'ReadContract',
  props: {
    addressContract: {
      type: String,
      default: '',
    },
    currentAccount: {
      type: String,
      default: '',
    },
    isConnect: {
      type: Boolean
    }
  },
  data() {
    return {
      totalSupply: 0,
      symbol: '',
      balance: '',
      provider: null
    }
  },
  watch: {
    isConnect(value) {
      if (value) {
        this.getErc20Info()
        this.initListeners()
        // listen for changes on an Ethereum address
        console.log(`listening for Transfer...`)
      } else {
        this.totalSupply = 0
        this.symbol = ''
        this.balance = ''
      }
    },
  },
  methods: {
    initListeners() {
      if(!window.ethereum) return
      if(!this.currentAccount) return
      this.provider = new ethers.providers.Web3Provider(window.ethereum)
      const erc20 = new ethers.Contract(this.addressContract, abi, this.provider)
      const fromMe = erc20.filters.Transfer(this.currentAccount, null)
      console.log('🚀 ~ file: ReadContract.vue ~ line 61 ~ initListeners ~ fromMe', fromMe)
      const toMe = erc20.filters.Transfer(null, this.currentAccount)
      console.log('🚀 ~ file: ReadContract.vue ~ line 63 ~ initListeners ~ toMe', toMe)

      this.provider.on(fromMe, (from, to, amount, event) => {
        console.log('Transfer|sent', { from, to, amount, event })
        this.queryTokenBalance()
      })

      this.provider.on(toMe, (from, to, amount, event) => {
        console.log('Transfer|received', { from, to, amount, event })
        this.queryTokenBalance()
      })
      
      this.$on('hook:beforeDestroy', () => {
        this.provider.removeAllListeners(toMe)
        this.provider.removeAllListeners(fromMe)
      })
    },
    async getErc20Info() {
      if (!window.ethereum) return

      const provider = new ethers.providers.Web3Provider(window.ethereum)
      const erc20 = new ethers.Contract(this.addressContract, abi, provider)
      try {
        const res = await erc20.symbol()
        this.symbol = res
      } catch (error) {
        console.log(
          '🚀 ~ file: ReadContract.vue ~ line 45 ~ getErc20Info ~ error',
          error
        )
      }
      try {
        const res = await erc20.totalSupply()
        this.totalSupply = ethers.utils.formatEther(res)
      } catch (error) {
        console.log(
          '🚀 ~ file: ReadContract.vue ~ line 50 ~ getErc20Info ~ error',
          error
        )
      }
    },
    async queryTokenBalance() {
      if (!window.ethereum) return
      if (!this.currentAccount) return

      const provider = new ethers.providers.Web3Provider(window.ethereum)
      const erc20 = new ethers.Contract(this.addressContract, abi, provider)
      try {
        const res = await erc20.balanceOf(this.currentAccount)
        this.balance = Number(ethers.utils.formatEther(res))
      } catch (error) {
        console.log(
          '🚀 ~ file: ReadContract.vue ~ line 62 ~ queryTokenBalance ~ error',
          error
        )
      }
    },
  },
}
</script>

<style></style>
