<template lang="pug">
q-page
  .row
    .col-12
      TransactionQR(:amount="amount")
      //- TransactionBrowser(:amount="amount")
  .row.q-col-gutter-md
    .col-6
      WalletMultiButton
    .col-6
      q-btn(@click="sendRandom" color="primary" label="send ramdon lamport")
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
      if (!publicKey.value) return;

      const transaction = new Transaction().add(
        SystemProgram.transfer({
          fromPubkey: publicKey.value,
          toPubkey: new PublicKey(process.env.STORE_ADDRESS_SOLANA.toString()),
          programId: new PublicKey(
            "G4a37Uh2bxak2teBAox3zbjQWT2Nr6xgD9jkj7Sgpm16"
          ),
          lamports: 1000000000,
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
<style scoped></style>
