<template>
  <q-page class="flex flex-center">
    <div class="q-pa-md full-width">
      <div class="row">
        <h2>Histórico</h2>
      </div>
      <div class="row">
        <q-list bordered class="rounded-borders col-12">
          <q-item
            v-for="calc in history"
            :key="calc.id"
            @click="showHistory(calc)"
            :to="link(calc)"
          >
            <q-item-section>
              <q-item-label v-if="calc.statement">{{ calc.statement }}</q-item-label>
              <q-item-label caption lines="2">
                {{calc.input}} / {{calc.weight}}
              </q-item-label>
            </q-item-section>
          </q-item>
        </q-list>
      </div>
    </div>
  </q-page>
</template>
<style scoped>
h2 {
  font-size: 21px;
  font-weight: bold;
}
</style>
<script>

import { mapMutations } from 'vuex';

export default {
  name: "History",
  data: function() {
    return {
      history: [],
    };
  },
  mounted() {
    this.loadHistory();
  },

  beforeRouteUpdate(to, from, next) {
    this.loadHistory();
    next();
  },
  watch: {
  },
  methods: {
    ...mapMutations(["CHANGE_HISTORY"]),
    link(history) {
      return "/history/" + encodeURIComponent(JSON.stringify(history));
    },

    showHistory(history) {
      this.CHANGE_HISTORY(history);
    },
    async loadHistory() {
      if (window.localStorage.history) {
        this.history = JSON.parse(window.localStorage.history);
      }
    }
  }
};
</script>
