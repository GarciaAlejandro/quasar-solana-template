<template lang="pug">

</template>

<script>
import SolanaWallets from "solana-wallets-vue";

// You can either import the default styles or create your own.
import "solana-wallets-vue/styles.css";
import { initWallet } from "solana-wallets-vue";
const walletOptions = {
  wallets: [
    new PhantomWalletAdapter(),
    new SlopeWalletAdapter(),
    new SolflareWalletAdapter({ network: "devnet" }),
  ],
  autoConnect: true,
};

export default {
  name: "wallet",
  mounted() {
    initWallet(wallet);
  },
};
</script>

<style></style>
