<template>
  <q-page class="flex flex-center">
    <div class="q-pa-md full-width">
      <div class="row">
        <h2>Exemplos</h2>
      </div>
      <div class="row">
        <q-list bordered class="rounded-borders col-12">
          <q-item v-for="example in examples"
            :key="example.id"
            :to="`/example/${example.id}`"
          >
            <q-item-section>
              <q-item-label>#{{example.id}}</q-item-label>
              <q-item-label caption lines="2">{{example.statement}}</q-item-label>
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
export default {
  name: "Examples",
  data: function() {
    return {
      examples: [],
    };
  },
  mounted() {
    this.loadExamples();
  },

  beforeRouteUpdate(to, from, next) {
    this.loadExamples();
    next();
  },
  watch: {
    examples() {
    }
  },
  methods: {
    async loadExamples() {
      const response = await fetch(process.env.API + '/api/example');
      if (response.ok) {
        this.examples = await response.json();
      }
    }
  }
};
</script>
