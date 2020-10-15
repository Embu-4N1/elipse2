<template>
  <q-page class="flex flex-center">
      <q-scroll-area class="fit q-pa-sm">
        <mathlive-mathfield
            :config="{smartFence:false}"
        >
            <!-- @focus="ping"
            :on-keystroke="displayKeystroke"
            v-model="formula" -->
            f(x)=
        </mathlive-mathfield>
      </q-scroll-area>
  </q-page>
</template>

<script>

export default {
  name: 'PageCalc'
}
</script>
