<template lang="pug">
h6 QR code styling for Vue
  .row.justify-between.q-col-gutter-md
    .col-sm-5.col-xs-12
      q-form(ref="generateTXForm")
        q-select(
          v-model="network",
          class="q-pb-lg"
          :rules="[(val) => !!val || 'The newtwork is required']"
          :options="networkOptions",
          label='Choose the network'
          outlined,
          rounded,
        )
        .text-body2.q-pl-md optional by default will be SOL
        q-input(
          v-model="splToken"
          rounded
          label="Address of SPL Token"
          outlined
        )
        .text-body2.q-pl-md The amount support decimals
        q-input(
          v-model="amount",
          rounded,
          label='Enter the amount to be charged '
          placeholder="Amount",
          outlined,
          :rules="[(val) => !!val || 'The amount is required']"
        )
        q-btn(
          label="Generate QR",
          noCaps,
          rounded,
          color="primary",
          @click="generateTrx()"
        )
    .col-sm-6.col-xs-12(style="background-color: #f5f5f5")
      .row.q-gutter-md
        .text-body2(v-show="url.reference")
          div reference generated: {{ url.reference }}
          div Amount: {{ amount }}
          div Network: {{ network }}
        div(
          v-show="resultTRX.url && resultTRX.status === 'success'"
        )
          div.text-body2 Payment validated successfully. The {{amount}} [Token] has been charged to the address STORE address
          a.text-body2(:href="resultTRX.url" target="_blank")
            .text-body2 Click to open the transaction
        div(v-if="resultTRX.status === 'success'" style="width: 24px; heigth: 24px; color:green;")
          svg.h-5.w-5(xmlns="http://www.w3.org/2000/svg", viewBox="0 0 20 20", fill="currentColor")
            path(fill-rule="evenodd", d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z", clip-rule="evenodd")
      #qr-code(v-show="resultTRX.status === 'pending'" ref="qrCode" class="q-py-sm")
      div#confirmed(
        v-show="resultTRX.status === 'confirmed'"
      )
        div.text-body2 The TRX was confirmed. Waiting the finalization of the transaction to be confirmed.
        q-circular-progress(
          show-value
          size="100px"
          :value="resultTRX.progress"
          :thickness="0.15"
          track-color="grey-5"
        ) {{ resultTRX.progress }}%
      q-btn(
        v-if="url.reference && resultTRX.status === 'pending'"
        class="q-mb-md"
        label="Cancel TRX",
        noCaps,
        rounded,
        color="primary",
        @click="cancelTRX()"
      )
</template>

<script>
import {
  Cluster,
  clusterApiUrl,
  Connection,
  PublicKey,
  SignatureStatus,
  Keypair,
} from "@solana/web3.js";
import {
  encodeURL,
  findTransactionSignature,
  LAMPORTS_PER_SOL,
  FindTransactionSignatureError,
  validateTransactionSignature,
  createQR,
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
      default: undefined,
    },
  },
  data() {
    const color = "#2a2a2a";
    const options = {
      width: 400,
      height: 400,
      type: "svg",
      image: `data:image/svg+xml;utf8,<svg fill="${encodeURIComponent(
        color
      )}" height="16" viewBox="0 0 16 14" width="16" xmlns="http://www.w3.org/2000/svg"><path d="m15.9176 11.038-2.6413 2.7713c-.0574.0602-.1269.1082-.2041.141s-.1604.0497-.2446.0497h-12.520966c-.059744 0-.118187-.0171-.168147-.0491-.0499596-.0321-.0892609-.0777-.1130745-.1313-.02381372-.0536-.03110249-.1129-.02097081-.1705.01013171-.0576.03724251-.111.07800141-.1538l2.6432769-2.7713c.05726-.06.12651-.1079.20346-.1407s.15996-.0498.2439-.05h12.52032c.0597 0 .1182.0171.1681.0492.05.032.0893.0776.1131.1313.0238.0536.0311.1128.021.1704-.0102.0576-.0373.1111-.078.1538zm-2.6413-5.58067c-.0574-.0602-.1269-.1082-.2041-.141s-.1604-.04971-.2446-.04966h-12.520966c-.059744 0-.118187.01708-.168147.04913-.0499596.03205-.0892609.07768-.1130745.13129-.02381372.0536-.03110249.11285-.02097081.17045.01013171.05761.03724251.11106.07800141.15379l2.6432769 2.77134c.05726.06004.12651.10794.20346.14073.07695.0328.15996.04979.2439.04993h12.52032c.0597 0 .1182-.01707.1681-.04913.05-.03205.0893-.07768.1131-.13129.0238-.0536.0311-.11285.021-.17045-.0102-.05761-.0373-.11106-.078-.15379zm-12.969666-1.99066h12.520966c.0842.00004.1674-.01687.2446-.04967s.1467-.0808.2041-.141l2.6413-2.771333c.0407-.042736.0678-.096189.078-.153792.0101-.057603.0028-.116847-.021-.170453s-.0631-.0992385-.1131-.1312911c-.0499-.0320526-.1084-.04912893-.1681-.0491309h-12.52032c-.08394.00013975-.16695.0171339-.2439.0499304s-.1462.0806976-.20346.1407366l-2.6425955 2.771333c-.0407196.04269-.0678184.09609-.07797306.15363-.01015467.05754-.00292373.11673.02080606.17031.0237297.05358.0629266.09922.1127835.13132.049857.03211.108207.04928.167893.04941z"/></svg>`,
      imageOptions: { hideBackgroundDots: true, imageSize: 0.15, margin: 8 },
      data: undefined,
      margin: 0,
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
        type: "extra-rounded",
        color: color,
      },
      backgroundOptions: {
        color: "#ffffff",
      },
      cornersSquareOptions: {
        color: color,
        type: "extra-rounded",
      },
      cornersDotOptions: {
        color: color,
        type: "square",
      },
    };
    const networks = ["devnet", "testnet", "mainnet-beta"];
    const verifiedOptions = ["confirmed", "finalized", "valid"];
    return {
      qr: undefined,
      options,
      extension: "svg",
      qrCode: undefined,
      connection: undefined,
      signature: undefined,
      network: "devnet",
      networkOptions: networks,
      verifiedOptions: verifiedOptions,
      splToken: undefined,
      resultTRX: {
        loading: false,
        status: "pending",
        url: undefined,
        maxValidations: 32,
        progress: 0,
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
      interval: undefined,
    };
  },
  methods: {
    /// MOBILE WALLET
    clearLastTRX() {
      this.url.recipient = undefined;
      this.url.amount = undefined;
      this.url.reference = undefined;
      this.url.label = undefined;
      this.url.message = undefined;
      this.url.memo = undefined;
      this.url.full = undefined;
      this.resultTRX.loading = false;
      this.resultTRX.status = "pending";
      this.resultTRX.url = undefined;
      this.resultTRX.validators = "confirmed";
      this.resultTRX.progress = 0;
    },
    cancelTRX() {
      this.clearLastTRX();
      this.splToken = undefined;
      clearInterval(this.interval);
      this.$q.notify({
        color: "negative",
        message: "Transaction cancelled",
        icon: "cancel",
        position: "top",
        timeout: 3000,
      });
    },
    async generateTrx() {
      this.clearLastTRX();
      if (await this.$refs.generateTXForm.validate()) {
        this.resultTRX.loading = true;
        console.log("------------------------------------");
        console.log("Generating transaction...");
        console.log("On network:", this.network);
        console.log("Amount:", this.amount);
        console.log("------------------------------------");
        this.connection = new Connection(
          clusterApiUrl(this.network),
          "confirmed"
        );
        await this.createQR();
        await this.paymentStatusQR();
        this.interval = setInterval(async () => {
          const { value } = await this.connection.getSignatureStatus(
            this.signature
          );
          console.log("📖 Confirmations: " + value.confirmations);
          if (value.confirmations !== null) {
            this.resultTRX.progress = Math.floor(
              (value.confirmations * 100) / this.resultTRX.maxValidations
            );
          } else {
            this.resultTRX.progress = 100;
          }
          if (this.resultTRX.progress === 100) {
            clearInterval(this.interval);
            await this.validateTrxQR();
          }
        }, 1250);
      }
    },
    createQR() {
      this.options.data = this.getURL();
      this.url.full = this.options.data;
      if (!(this.qrCode instanceof QRCodeStyling)) {
        this.qrCode = new QRCodeStyling(this.options);
        this.qrCode.append(this.$refs["qrCode"]);
      } else {
        this.qrCode.update(this.options);
      }
    },
    getURL() {
      this.url.recipient = new PublicKey(process.env.STORE_ADDRESS_SOLANA);
      this.url.amount = new BigNumber(this.amount);
      this.url.reference = new Keypair().publicKey;
      this.url.label = "Quasar Solana store";
      this.url.message = "AG store - first order";
      this.url.memo = "AGC#12#FistTRX";
      let obj = {
        recipient: this.url.recipient,
        amount: this.url.amount,
        reference: this.url.reference,
        label: this.url.label,
        message: this.url.message,
        memo: this.url.memo,
      };
      if (this.splToken) {
        const splToken = new PublicKey(this.splToken);
        obj.splToken = splToken;
      }
      return encodeURL(obj);
    },
    async paymentStatusQR() {
      console.log("\n Find the transaction");
      let signatureInfo;
      this.resultTRX.status = "pending";

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
        this.interval = setInterval(async () => {
          console.count("Checking for transaction...");
          try {
            signatureInfo = await findTransactionSignature(
              this.connection,
              this.url.reference,
              undefined,
              "confirmed"
            );
            console.log(signatureInfo);
            console.log("\n 🖌  Signature found: ", signatureInfo.signature);
            this.$q.notify({
              message: "🖌 Signature found 💰" + signatureInfo.signature,
              color: "green",
              icon: "done_all",
            });
            this.signature = signatureInfo.signature;
            clearInterval(this.interval);
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
      this.resultTRX.status = "confirmed";
    },
    async validateTrxQR() {
      console.log("\n 🔗 Validate transaction \n");
      this.$q.notify({
        message: "🔗 Validating transaction...",
      });
      try {
        const response = await validateTransactionSignature(
          this.connection,
          this.signature,
          this.url.recipient,
          this.url.amount,
          undefined,
          this.url.reference,
          "finalized"
        );
        console.log(response);
        // Update payment status
        this.resultTRX.status = "validated";
        this.$q.notify({
          message:
            "✅ Payment validated. The " + this.amount + " was send correctly",
        });
        console.log("✅ Payment validated");
        let url = "https://solscan.io/tx/" + this.signature;
        if (this.network === "testnet" || this.network === "devnet") {
          url = url + "?cluster=" + this.network;
        }
        console.log(url);
        this.resultTRX.url = url;
        this.resultTRX.status = "success";
        this.resultTRX.loading = false;
      } catch (error) {
        this.$q.notify({
          message: "❌ Transaction not validated",
          color: "red",
          icon: "error",
        });
        console.error("❌ Payment failed", error);
      }
    },
  },
};
</script>

<style lang="css" scoped>
.qr-code {
  max-width: 300px;
  max-height: 300px;
  margin: 0 auto;
}
</style>
