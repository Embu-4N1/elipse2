<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="leftDrawerOpen = !leftDrawerOpen"
        />

        <q-toolbar-title>
          Elipse
        </q-toolbar-title>

        <div></div>
      </q-toolbar>
    </q-header>

    <q-drawer
      v-model="leftDrawerOpen"
      show-if-above
      bordered
      content-class="bg-grey-1"
    >
      <q-list>
        <EssentialLink
          v-for="link in essentialLinks"
          :key="link.title"
          v-bind="link"
        />
      </q-list>
    </q-drawer>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import EssentialLink from "components/EssentialLink.vue";

const linksData = [
  {
    title: "Home",
    // caption: '/',
    icon: "home",
    link: "/"
  },
  {
    title: "Exemplos",
    // caption: 'exemplos',
    icon: "book",
    link: "/#/examples"
  }
];

export default {
  name: "MainLayout",
  components: { EssentialLink },
  data() {
    return {
      leftDrawerOpen: false,
      essentialLinks: linksData,
      computed: {
        example() {
          return $route.params.example;
        }
      }
    };
  },
  beforeRouteUpdate(to, from, next) {
    // this.loadExample(this.exampleId);
    next();
  },
  meta: {
    noscript: {
      default: "Enable javascript, please with cheese!"
    }
  }
};
</script>
