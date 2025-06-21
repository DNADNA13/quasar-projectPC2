<template>
  <q-page class="flex flex-center">
    <CurrencyConverter />
  </q-page>
</template>

<script>
import CurrencyConverter from 'src/components/currency/CurrencyConverter.vue'

export default {
  name: 'CurrencyPage',
  components: {
    CurrencyConverter,
  },
}
</script>
