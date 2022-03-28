<template lang="pug">
q-layout(view="lHh Lpr lFf")
  q-header(elevated)
    q-toolbar
      q-btn(
        flat
        dense
        round
        icon="menu"
        aria-label="Menu"
        @click="toggleLeftDrawer"
      )

      q-toolbar-title Quasar App [Solana]
      div Quasar v{{ $q.version }}

  q-drawer(v-model="leftDrawerOpen" show-if-above bordered)
    q-list
      q-item-label(header) Essential Links

      EssentialLink(
        v-for="link in essentialLinks"
        :key="link.title"
        v-bind="link"
      )
  q-page-container
    router-view
</template>

<script>
import EssentialLink from "components/EssentialLink.vue";

const linksList = [
  {
    title: "Docs",
    caption: "quasar.dev",
    icon: "school",
    link: "https://quasar.dev",
  },
  {
    title: "Github",
    caption: "github.com/GarciaAlejandro",
    icon: "code",
    link: "https://github.com/GarciaAlejandro",
  },
];

import { defineComponent, ref } from "vue";

export default defineComponent({
  name: "MainLayout",

  components: {
    EssentialLink,
  },

  setup() {
    const leftDrawerOpen = ref(false);

    return {
      essentialLinks: linksList,
      leftDrawerOpen,
      toggleLeftDrawer() {
        leftDrawerOpen.value = !leftDrawerOpen.value;
      },
    };
  },
});
</script>
