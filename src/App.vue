<template>
  <div class="container">
    <h1>Polkadot-JS Signer</h1>
    <!-- A select that display all accounts that can be selected  -->
    <div>

      <p>Add an account</p>
      <textarea v-model="accountMnemonic"/>
      <br/>
      <button @click="addAccount">Add account</button>
      <hr/>

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
        <p>Hex Message</p>
        <textarea class="message" :value="computedMessage" disabled/>
        <br/>
        <hr/>
      </div>

      <!-- A button that sign a message with the selected account -->
      <p>Sign a message with the selected account</p>
      <button @click="signMessage">Sign message</button>

      <!-- A input that display the signed message -->
      <p>Signed message</p>
      <textarea class="signedMessage" :value="signedMessage" disabled/>
    </div>
  </div>

</template>

<script setup lang="ts">
import {encodeAddress, decodeAddress} from '@polkadot/util-crypto'
import {u8aToHex} from "@polkadot/util";
import {ref, onMounted, Ref, computed} from "vue";
import {Keyring} from '@polkadot/ui-keyring';
import {cryptoWaitReady} from '@polkadot/util-crypto';
import {KeyringAddress} from "@polkadot/ui-keyring/types";
import {stringToHex} from '@polkadot/util';

const accounts: Ref<KeyringAddress[]> = ref([]);
const selectedAccount: Ref<KeyringAddress | undefined> = ref();

const message: Ref<string> = ref('');
const signedMessage: Ref<string> = ref('');
const accountMnemonic: Ref<string> = ref('');

const keyring = new Keyring();
keyring.loadAll({ss58Format: 195, type: 'sr25519'});

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

const computedMessage = computed(() => {
  return stringToHex(message.value);
});

const addAccount = async () => {
  keyring.addUri(accountMnemonic.value);
  accountMnemonic.value = '';
}

const connect = async () => {
  await cryptoWaitReady();

  keyring.accounts.subject.subscribe(() => {
    accounts.value = keyring.getAccounts();

    if (!selectedAccount.value) {
      selectedAccount.value = accounts.value[0];
    }
  });

};

onMounted(connect);

async function signMessage() {
  const address = selectedAccount.value?.address;
  if (!address) {
    throw new Error('No address');
  }

  const pair = keyring.getPair(address);
  pair.unlock();
  const signResult = pair.sign(stringToHex(message.value))

  signedMessage.value = u8aToHex(signResult);
}

</script>

<style scoped>

textarea {
  width: 100%;
  height: 60px;
}

</style>
