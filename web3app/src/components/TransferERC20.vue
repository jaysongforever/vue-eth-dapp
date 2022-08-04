<template>
  <div>
    发送数量：
    <input :disabled="!isConnect" type="text" :value="transValue" @input="transChange" />
    目标地址：
    <input :disabled="!isConnect" type="text" :value="addressValue" @input="addressChange" />
    <button :disabled="!isConnect" @click="transfer">发送代币</button>
  </div>
</template>

<script>
import { ERC20ABI as abi } from '../abi/ERC20ABI'
import { ethers } from 'ethers'

export default {
  name: 'TransferERC20',
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
    },
  },
  data() {
    return {
      transValue: '',
      addressValue: '',
    }
  },
  watch: {
    isConnect(value) {
      if (!value) {
        this.transValue = ''
        this.addressValue = ''
      }
    },
  },
  methods: {
    async transfer() {
      if (this.transValue && this.addressValue && this.isConnect) {
        if (!window.ethereum) return
        const provider = new ethers.providers.Web3Provider(window.ethereum)
        const signer = provider.getSigner()
        const erc20 = new ethers.Contract(this.addressContract, abi, signer)

        try {
          const tr = await erc20.transfer(this.addressValue, ethers.utils.parseEther(this.transValue))
          console.log(`TransactionResponse TX hash: ${tr.hash}`)
          const receipt = await tr.wait()
          if (receipt ) {
            this.transValue = ''
            this.addressValue = ''
          }
          console.log('transfer receipt', receipt)
        } catch (error) {
          console.log(error)
        }
      }
    },
    transChange(e) {
      this.transValue = e.target.value
    },
    addressChange(e) {
      this.addressValue = e.target.value
    },
  },
}
</script>

<style></style>
