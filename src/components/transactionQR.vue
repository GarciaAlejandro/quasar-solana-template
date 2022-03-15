<template lang="pug">
h6 QR code styling for Vue
.row(style='width: 35%')
  .col-12
    q-input(v-model="amount" debounce="500" placeholder="Amount" outlined)
  .col-12
    div reference : {{url.reference}}
    div(id='qr-code' ref='qrCode')

</template>

<script>
import {
  Cluster,
  clusterApiUrl,
  Connection,
  PublicKey,
  Keypair,
} from "@solana/web3.js";
import {
  encodeURL,
  findTransactionSignature,
  LAMPORTS_PER_SOL,
  FindTransactionSignatureError,
  validateTransactionSignature,
} from "@solana/pay";
import BigNumber from "bignumber.js";
import QRCodeStyling, {
  DrawType,
  TypeNumber,
  Mode,
  ErrorCorrectionLevel,
  DotType,
  CornerSquareType,
  CornerDotType,
  Extension,
} from "qr-code-styling";

export default {
  name: "TransactionQR",
  props: {
    amount: {
      type: String,
      default: "0",
    },
  },
  data() {
    const options = {
      width: 300,
      height: 300,
      type: "svg",
      data: undefined,
      image: "/favicon.ico",
      margin: 5,
      qrOptions: {
        typeNumber: 0,
        mode: "Byte",
        errorCorrectionLevel: "Q",
      },
      imageOptions: {
        hideBackgroundDots: true,
        imageSize: 0.4,
        margin: 5,
        crossOrigin: "anonymous",
      },
      dotsOptions: {
        color: "black",
        type: "rounded",
      },
      backgroundOptions: {
        color: "#ffffff",
      },
      cornersSquareOptions: {
        color: "#35495E",
        type: "extra-rounded",
      },
      cornersDotOptions: {
        color: "#00FFA4",
        type: "dot",
      },
    };
    return {
      qr: undefined,
      options,
      extension: "svg",
      qrCode: undefined,
      connection: new Connection(clusterApiUrl("mainnet-beta"), "confirmed"),
      signature: undefined,
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
      // Mobile wallet
      await this.createQR("update");
      await this.paymentStatusQR();
      await this.validateTrxQR();
    },
  },
  mounted() {
    this.createQR("create");
  },
  methods: {
    /// MOBILE WALLET
    createQR(state) {
      this.options.data = this.getURL();
      this.url.full = this.options.data;
      if (state === "create") {
        this.qrCode = new QRCodeStyling(this.options);
        this.qrCode.append(this.$refs.qrCode);
      } else {
        this.qrCode.update(this.options);
      }
    },
    getURL() {
      console.log("2. 🛍 Simulate a customer checkout \n");
      this.url.recipient = new PublicKey(process.env.STORE_ADDRESS_SOLANA);
      this.url.amount = new BigNumber(this.amount);
      this.url.reference = new Keypair().publicKey;
      this.url.label = "Quasar vue store";
      this.url.message = "AG store - first order";
      this.url.memo = "AGC#12#FistTRX";
      console.log("🛍 URL: ", this.url);
      const splToken = new PublicKey(process.env.SPL_TOKEN_ADDRESS_USDC);

      return encodeURL({
        recipient: this.url.recipient,
        amount: this.url.amount,
        reference: this.url.reference,
        // splToken: splToken,
        label: this.url.label,
        message: this.url.message,
        memo: this.url.memo,
      });
    },
    async paymentStatusQR() {
      console.log("\n5. Find the transaction");
      let signatureInfo;
      let paymentStatus = "pending";

      const { signature } = await new Promise((resolve, reject) => {
        /**
         * Retry until we find the transaction
         *
         * If a transaction with the given reference can't be found, the `findTransactionSignature`
         * function will throw an error. There are a few reasons why this could be a false negative:
         *
         * - Transaction is not yet confirmed
         * - Customer is yet to approve/complete the transaction
         *
         * You can implement a polling strategy to query for the transaction periodically.
         */
        const interval = setInterval(async () => {
          console.count("Checking for transaction...");
          try {
            signatureInfo = await findTransactionSignature(
              this.connection,
              this.url.reference,
              undefined,
              "confirmed"
            );
            console.log("\n 🖌  Signature found: ", signatureInfo.signature);
            this.$q.notify({
              message: "Transaction found 💰 " + signatureInfo.signature,
              color: "green",
              icon: "done_all",
            });
            this.signature = signatureInfo.signature;
            clearInterval(interval);
            resolve(signatureInfo);
          } catch (error) {
            if (!(error instanceof FindTransactionSignatureError)) {
              console.error(error);
              clearInterval(interval);
              reject(error);
            }
          }
        }, 1250);
      });
      // Update payment status
      paymentStatus = "confirmed";
    },
    async validateTrxQR() {
      console.log("\n6. 🔗 Validate transaction \n");
      try {
        await validateTransactionSignature(
          this.connection,
          this.signature,
          this.url.recipient,
          this.url.amount,
          undefined,
          this.url.reference
        );

        // Update payment status
        let paymentStatus = "validated";
        console.log("✅ Payment validated");
        console.log("📦 Ship order to customer");
        this.$q.notify({
          message: "Payment validated 💰 " + this.signature,
          color: "green",
          icon: "done_all",
        });
      } catch (error) {
        this.$q.notify({
          message: "Transaction not validated",
          color: "red",
          icon: "error",
        });
        console.error("❌ Payment failed", error);
      }
    },
  },
};
</script>

<style></style>
