<template>
  <div class="q-pa-md">
    <q-form @submit.prevent="convertirMoneda">
      <q-input v-model.number="amount" type="number" label="Monto" class="q-mb-md" required />

      <div class="row q-gutter-md q-mb-md">
        <q-select v-model="from" :options="monedas" label="Desde" emit-value map-options />
        <q-select v-model="to" :options="monedas" label="Hacia" emit-value map-options />
        <q-btn label="⇄" @click="intercambiar" flat round />
      </div>

      <q-btn type="submit" label="Convertir" color="primary" class="q-mb-md" />

      <div v-if="resultado" class="q-mt-md text-h6">
        {{ resultado }}
      </div>
    </q-form>
  </div>
</template>

<script>
export default {
  name: 'CurrencyConverter',
  data() {
    return {
      amount: null,
      from: '',
      to: '',
      monedas: [],
      resultado: '',
    }
  },
  mounted() {
    this.cargarMonedas()
  },
  methods: {
    async cargarMonedas() {
      try {
        const response = await this.$axios.get('https://api.frankfurter.app/currencies')
        this.monedas = Object.entries(response.data).map(([code, name]) => ({
          label: `${code} - ${name}`,
          value: code,
        }))
      } catch {
        // <-- **CAMBIO AQUÍ: Eliminamos 'error'**
        this.$q.notify({
          type: 'negative',
          message: 'Error al cargar las monedas',
        })
      }
    },
    async convertirMoneda() {
      if (!this.amount || !this.from || !this.to) {
        this.$q.notify({ type: 'warning', message: 'Completa todos los campos' })
        return
      }

      try {
        const url = `https://api.frankfurter.app/latest?amount=${this.amount}&from=${this.from}&to=${this.to}`
        const response = await this.$axios.get(url)
        const conversion = response.data.rates[this.to]
        this.resultado = `${this.amount} ${this.from} equivalen a ${conversion} ${this.to}`
      } catch {
        // <-- **CAMBIO AQUÍ: Eliminamos 'error'**
        this.$q.notify({ type: 'negative', message: 'Error en la conversión' })
      }
    },
    intercambiar() {
      ;[this.from, this.to] = [this.to, this.from]
    },
  },
}
</script>
