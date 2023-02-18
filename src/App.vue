<template>
  <div class="container">
    <h1>Polkadot-JS Signer</h1>
    <!-- A select that display all accounts that can be selected  -->
    <div>
      <p>Select the account to sign a message </p>
      <select v-model="selectedAccount">
        <option v-for="account in accounts" :key="account.address" :value="account">
          {{ account.meta.name }} ({{ account.address }})
        </option>
      </select>

      <p>Public Key: {{ publicKey }}</p>
      <p>Efinity Address: {{ efinityAddress }}</p>

      <!-- A input with the message that will be signed-->
      <p>Message to sign</p>
      <textarea class="message" v-model="message"/>

      <!-- A button that sign a message with the selected account -->
      <p>Sign a message with the selected account</p>
      <button @click="signMessage">Sign message</button>

      <!-- A input that display the signed message -->
      <p>Signed message</p>
      <textarea class="signedMessage" :value="signedMessage"/>
    </div>
  </div>

</template>

<script setup lang="ts">
import {web3Accounts, web3Enable, web3FromSource} from '@polkadot/extension-dapp';
import {encodeAddress, decodeAddress, mnemonicGenerate} from '@polkadot/util-crypto'
import {stringToHex, u8aToHex} from "@polkadot/util";
import {ref, onMounted, Ref, computed} from "vue";
import {InjectedAccountWithMeta} from "@polkadot/extension-inject/types";

const accounts: Ref<InjectedAccountWithMeta[]> = ref([]);
const selectedAccount: Ref<InjectedAccountWithMeta | undefined> = ref();
const message: Ref<string> = ref('');

const signedMessage: Ref<string> = ref('');

const publicKey = computed(() => {
  if (!selectedAccount.value) {
    return '';
  }

  return u8aToHex(decodeAddress(selectedAccount.value.address));
});

const efinityAddress = computed(() => {
  if (!selectedAccount.value) {
    return '';
  }

  return encodeAddress(publicKey.value, 195);
});

const connect = async () => {
  const extensions = await web3Enable('Polkadot-JS Signer');
  if (extensions.length === 0) {
    throw new Error('no extension');
  }

  accounts.value = await web3Accounts();
  selectedAccount.value = accounts.value[0];
};

onMounted(connect);

async function signMessage() {
  const source = selectedAccount.value?.meta.source;
  if (!source) {
    throw new Error('No source');
  }
  if (!selectedAccount.value) {
    throw new Error('No account selected');
  }

  const injector = await web3FromSource(source);
  const signRaw = injector?.signer?.signRaw;

  if (!signRaw) {
    throw new Error('No signRaw function');
  }

  const {signature} = await signRaw({
    address: selectedAccount.value.address,
    data: stringToHex(message.value),
    type: 'bytes'
  });

  signedMessage.value = signature;
}

</script>

<style scoped>

.message, .signedMessage {
  width: 100%;
  height: 60px;
}

</style>
