<template lang="pug">
q-page
  .q-px-xl
    .row.q-col-gutter-md.q-pt-md
      .col-sm-6.col-xs-12
        WalletMultiButton
      .col-sm-6.col-xs-12
        .text-body2 Click on the button to send lamports to random account
        .text-body2 [This feature is connected to devnet and is required the account connected ]
        q-btn(
          class="q-my-md"
          @click="sendRandom",
          noCaps,
          color="primary",
          label="send 0.01 SOL to Store Address"
        )
    hr
    .row
      .col-12
        TransactionQR()
    hr
    .row.q-py-md
      .col-12
        TransactionBrowser(:amount="amount")
</template>

<script>
import { defineComponent } from "vue";
import { WalletMultiButton } from "solana-wallets-vue";
import SolanaWallets from "solana-wallets-vue";
import "solana-wallets-vue/styles.css";
import { initWallet } from "solana-wallets-vue";
import {
  PhantomWalletAdapter,
  SlopeWalletAdapter,
  SolflareWalletAdapter,
} from "@solana/wallet-adapter-wallets";
const walletOptions = {
  wallets: [
    new PhantomWalletAdapter(),
    new SlopeWalletAdapter(),
    new SolflareWalletAdapter({ network: "devnet" }),
  ],
  autoConnect: false,
};
import { useWallet } from "solana-wallets-vue";
import {
  Connection,
  clusterApiUrl,
  Keypair,
  SystemProgram,
  PublicKey,
  Transaction,
} from "@solana/web3.js";
import TransactionQR from "../components/transactionQR.vue";
import TransactionBrowser from "../components/transactionBrowser.vue";
export default defineComponent({
  name: "PageIndex",
  components: {
    WalletMultiButton,
    TransactionQR,
    TransactionBrowser,
  },
  data() {
    return {
      amount: "0.00000",
    };
  },
  beforeMount() {
    initWallet(walletOptions);
  },
  methods: {
    async sendRandom() {
      const connection = new Connection(clusterApiUrl("devnet"));
      const { publicKey, sendTransaction } = useWallet();
      if (!publicKey.value) {
        this.$q.notify({
          color: "red-7",
          message: "Please connect a wallet",
        });
        return;
      }

      const transaction = new Transaction().add(
        SystemProgram.transfer({
          fromPubkey: publicKey.value,
          toPubkey: new PublicKey(process.env.STORE_ADDRESS_SOLANA.toString()),
          programId: new PublicKey(
            "G4a37Uh2bxak2teBAox3zbjQWT2Nr6xgD9jkj7Sgpm16"
          ),
          lamports: 10000000,
        })
      );
      try {
        const signature = await sendTransaction(transaction, connection);
        console.log(signature);
        const confirm = await connection.confirmTransaction(
          signature,
          "processed"
        );
        this.$q.notify({
          message: signature,
        });
      } catch (error) {
        this.$q.notify({
          message: error.message,
        });
      }
      console.log(confirm);
    },
  },
});
</script>
<style lang="css" scoped>
hr {
  border: 0;
  border-top: 1.5px solid #eee;
}
</style>
