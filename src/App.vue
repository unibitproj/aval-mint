<template>
  <div>
    <h1>AVAL MINT TOOL</h1>
    <p>地址: {{ address }}</p>
    <p>私钥：<input type="password" v-model="privateKey" /></p>
    <p>rpc：<input type="text" v-model="web3Rpc" /></p>
    <p>gas最高价格(gwei)：<input type="text" v-model="gasCap" /></p>
    <p>gas小费(gwei)：<input type="text" v-model="gasTip" /></p>
    <p>张数：<input type="text" v-model="count" /></p>
    <h2>
      <span v-if="doing" class="deploy" @click="start">Mint中</span>
      <span v-else class="deploy" @click="start">开始</span>
    </h2>
    <h3>mint要输入私钥，请尽量用小号私钥填充，代码开源在<a href="https://github.com/minchenzz/aval-mint" target="_blank">github</a>，如担心安全问题请自行下载并部署该服务</h3>
    <p>author: <a href="https://twitter.com/chenmin22998595" target="_blank">@chems</a></p>
    <ul>
      <li v-for="tx in confirmedTxHashs">已完成交易：<a :href="'https://polygonscan.com/tx/'+tx" target="_blank">{{ tx }}</a></li>
    </ul>

  </div>

</template>

<script setup>
import {ref} from "vue";
import { ethers, JsonRpcProvider, Wallet, formatEther, parseUnits } from "ethers";

const address = ref('')
const privateKey = ref('')
const web3Rpc = ref('https://avalanche.drpc.org')
const gasCap = ref('500')
const gasTip = ref('5')
const count = ref(1)
const doing = ref(false)
const confirmedTxHashs = ref([])

const data = '0x646174613a2c7b2270223a226173632d3230222c226f70223a226d696e74222c227469636b223a226176616c222c22616d74223a22313030303030303030227d'

let providerInstance

let currentNonce

const doTx = async () => {
  let tx = {
    from: address.value,
    to: address.value,
    data: data,
    maxFeePerGas: parseUnits(gasCap.value, 'gwei'),
    maxPriorityFeePerGas: parseUnits(gasTip.value, 'gwei'),
    gasLimit: 22024,
    value: 0,
    nonce: currentNonce,
    chainId: 43114
  }
  const wallet = new Wallet(privateKey.value, providerInstance);
  const txResp = await wallet.sendTransaction(tx);
  console.log('txhash', txResp.hash)
  console.log(tx)
  try {
    const receipt = await providerInstance.waitForTransaction(txResp.hash);
    console.log('tx receipt', receipt)
    confirmedTxHashs.value.push(txResp.hash.toString())
  } catch (e) {
    console.log(e)
  }
  currentNonce++
}

const start2 = async () => {
    providerInstance = new JsonRpcProvider(web3Rpc.value);
  const wallet = new ethers.Wallet(privateKey.value, providerInstance);
  address.value = wallet.address
  currentNonce = await providerInstance.getTransactionCount(address.value);
  for (let i = 0; i < count.value; i++) {
    await doTx()
  }
}


const start = async () => {
  if (doing.value) {
    return
  }
  doing.value = true

  try {
    await start2()
  } catch (e) {
    console.log(e)
  }
  doing.value = false
}




</script>

<style scoped>

.deploy {
  cursor: pointer;
}


</style>
