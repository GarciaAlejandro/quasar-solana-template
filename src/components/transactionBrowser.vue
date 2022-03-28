<template lang="pug">
h6 Using solana pay button [coming soon]
//- .row.q-col-gutter-md.justify-between
//-   .col-sm-4.col-xs-12
//-     q-input(v-model="amount" rounded dense debounce="500" placeholder="Amount" outlined)
//-   .col-sm-6.col-xs-12
//-     div reference : {{url.reference}}
//-     div(id='qr-code' ref='qrCode')

</template>

<script>
import {
  Cluster,
  clusterApiUrl,
  Connection,
  PublicKey,
  Keypair,
  Signer,
  sendAndConfirmTransaction,
  SystemProgram,
} from "@solana/web3.js";
import { encodeURL, createTransaction, parseURL } from "@solana/pay";
import BigNumber from "bignumber.js";
import { useWallet } from "solana-wallets-vue";
import QrcodeScanner from "./qrcode-scanner.vue";
export default {
  name: "TransactionQR",
  components: {
    QrcodeScanner,
  },
  props: {
    amount: {
      type: String,
      default: "0",
    },
  },
  data() {
    return {
      connection: new Connection(clusterApiUrl("devnet"), "confirmed"),
      signature: undefined,
      payer: undefined,
      userInfo: {
        publicKey: undefined,
      },
      walletProvider: {
        sendTransaction: undefined,
        signature: undefined,
      },
      qrscanner: {
        text: undefined,
        result: undefined,
      },
      url: {
        recipient: undefined,
        amount: undefined,
        reference: undefined,
        label: undefined,
        message: undefined,
        memo: undefined,
        full: undefined,
      },
    };
  },
  watch: {
    async amount(newVal) {
      // Desktop wallet
      const { publicKey, sendTransaction } = useWallet();
      this.publicKey = publicKey;
      this.sendTransaction = sendTransaction;
      if (!publicKey.value) return;
      await this.payUsingWallet();
    },
  },
  methods: {
    getURL() {
      this.url.recipient = new PublicKey(process.env.STORE_ADDRESS_SOLANA);
      this.url.amount = new BigNumber(this.amount);
      this.url.reference = new Keypair().publicKey;
      this.url.label = "Quasar vue store";
      this.url.message = "Jungle Cats store - your order - #001234";
      this.url.memo = "JC#4098";
      // console.log("🛍 URL: ", this.url);
      return encodeURL({
        recipient: this.url.recipient,
        amount: this.url.amount,
        reference: this.url.reference,
        label: this.url.label,
        message: this.url.message,
        memo: this.url.memo,
      });
    },
    async payUsingWallet() {
      // let url = this.getURL();
      let url =
        "solana:mvines9iiHiQTysrwkJjGf2gb9Ex9jXJX8ns3qwf2kN?amount=0.01&spl-token=EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v&label=Michael&message=Thanks%20for%20all%20the%20fish&memo=OrderId5678";
      const { recipient, amount, splToken, reference, label, message, memo } =
        parseURL(url);

      const tx = await createTransaction(
        this.connection,
        this.publicKey.value,
        recipient,
        amount,
        {
          reference: reference,
          memo: memo,
        }
      );
      const signature = await this.sendTransaction(tx, this.connection);
      const confirm = await this.connection.confirmTransaction(
        signature,
        "processed"
      );
      console.log("💰 Confirm: ", confirm);
    },
    onScan(decodedText, decodedResult) {
      this.qrscanner.text = decodedText;
      this.qrscanner.result = decodedResult;
      console.log("🔍 QR code: ", decodedText);
      console.log("🔍 QR result: ", decodedResult);
    },
  },
};
</script>

<style></style>
