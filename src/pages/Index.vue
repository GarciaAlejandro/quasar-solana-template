<template lang="pug">
q-page.flex.flex-center
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

// You can either import the default styles or create your own.
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
export default defineComponent({
  name: "PageIndex",
  components: { WalletMultiButton },
  beforeMount() {
    initWallet(walletOptions);
  },
  methods: {
    async sendRandom() {
      const connection = new Connection(clusterApiUrl("devnet"));
      console.log(connection);
      const { publicKey, sendTransaction } = useWallet();
      if (!publicKey.value) return;
      // s = "GyoixxRkQ99CfV6twBah68AfxCisYhwyVuUxek8VsNFy";
      // var result = [];
      // for (var i = 0; i < s.length; i += 2) {
      //   result.push(parseInt(s.substring(i, i + 2), 16));
      // }
      // result = Uint8Array.from(result);
      // console.log(result);

      const transaction = new Transaction().add(
        SystemProgram.transfer({
          fromPubkey: publicKey.value,
          toPubkey: new PublicKey(
            "GyoixxRkQ99CfV6twBah68AfxCisYhwyVuUxek8VsNFy"
          ),
          programId: new PublicKey(
            "G4a37Uh2bxak2teBAox3zbjQWT2Nr6xgD9jkj7Sgpm16"
          ),
          lamports: 1,
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
