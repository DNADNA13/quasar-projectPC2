<template>
  <div class="q-pa-md" style="max-width: 500px; margin: auto">
    <q-card class="q-pa-md q-shadow--2 rounded-borders">
      <q-card-section class="text-center q-pb-md">
        <div class="text-h5 text-weight-bold flex items-center justify-center">
          <q-icon name="trending_up" class="q-mr-sm" color="green-6" size="md" />
          Conversión de Monedas
        </div>
      </q-card-section>

      <q-form @submit.prevent="convertirMoneda">
        <div class="q-mb-md">
          <q-item-label class="q-mb-sm text-subtitle1 text-weight-medium"
            >Monto a convertir</q-item-label
          >
          <q-input
            v-model.number="amount"
            type="number"
            outlined
            class="q-mb-md"
            required
            min="0"
          />
        </div>

        <div class="row q-col-gutter-md q-mb-md items-center">
          <div class="col-5">
            <q-item-label class="q-mb-sm text-subtitle1 text-weight-medium">Desde</q-item-label>
            <q-select
              v-model="from"
              :options="monedas"
              label="Seleccionar Moneda"
              outlined
              emit-value
              map-options
              option-label="label"
              option-value="value"
            >
              <template v-slot:selected-item="scope">
                <q-item>
                  <q-item-section>
                    <q-item-label class="text-subtitle1 text-weight-bold">{{
                      scope.opt.value
                    }}</q-item-label>
                    <q-item-label caption>{{ scope.opt.name }}</q-item-label>
                  </q-item-section>
                </q-item>
              </template>
              <template v-slot:option="scope">
                <q-item v-bind="scope.itemProps" v-on="scope.itemHandlers">
                  <q-item-section>
                    <q-item-label>{{ scope.opt.value }} - {{ scope.opt.name }}</q-item-label>
                  </q-item-section>
                </q-item>
              </template>
            </q-select>
          </div>

          <div class="col-2 text-center">
            <q-btn
              icon="swap_vert"
              @click="intercambiar"
              flat
              round
              size="lg"
              color="grey-7"
              class="q-mt-md"
            />
          </div>

          <div class="col-5">
            <q-item-label class="q-mb-sm text-subtitle1 text-weight-medium">Hacia</q-item-label>
            <q-select
              v-model="to"
              :options="monedas"
              label="Seleccionar Moneda"
              outlined
              emit-value
              map-options
              option-label="label"
              option-value="value"
            >
              <template v-slot:selected-item="scope">
                <q-item>
                  <q-item-section>
                    <q-item-label class="text-subtitle1 text-weight-bold">{{
                      scope.opt.value
                    }}</q-item-label>
                    <q-item-label caption>{{ scope.opt.name }}</q-item-label>
                  </q-item-section>
                </q-item>
              </template>
              <template v-slot:option="scope">
                <q-item v-bind="scope.itemProps" v-on="scope.itemHandlers">
                  <q-item-section>
                    <q-item-label>{{ scope.opt.value }} - {{ scope.opt.name }}</q-item-label>
                  </q-item-section>
                </q-item>
              </template>
            </q-select>
          </div>
        </div>

        <q-btn type="submit" label="Convertir" color="primary" class="full-width q-py-sm" rounded />

        <div v-if="resultado" class="q-mt-lg text-center text-h6 text-weight-medium">
          {{ resultado }}
        </div>
      </q-form>
    </q-card>
  </div>
</template>

<script>
import { Notify } from 'quasar'
export default {
  name: 'CurrencyConverter',
  data() {
    return {
      amount: null,
      from: null, 
      to: null, 
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
          name: name, 
        }))

        
        if (!this.from && this.monedas.some((m) => m.value === 'USD')) {
          this.from = 'USD'
        }
        if (!this.to && this.monedas.some((m) => m.value === 'EUR')) {
          this.to = 'EUR'
        }
      } catch {
        Notify.create({
          type: 'negative',
          message: 'Error al cargar las monedas',
        })
      }
    },
    async convertirMoneda() {
      const monto = Number(this.amount)
      
      if (isNaN(monto) || monto <= 0) {
        Notify.create({ type: 'negative', message: 'Ingresa un monto válido mayor a cero.' })
        return
      }
      if (!this.from || !this.to) {
        Notify.create({ type: 'negative', message: 'Selecciona ambas monedas.' })
        return
      }

      try {
        const url = `https://api.frankfurter.app/latest?amount=${this.amount}&from=${this.from}&to=${this.to}`
        const response = await this.$axios.get(url)

        if (response.data.rates && response.data.rates[this.to]) {
          const conversion = response.data.rates[this.to]
          
          const fromName = this.monedas.find((m) => m.value === this.from)?.name || this.from
          const toName = this.monedas.find((m) => m.value === this.to)?.name || this.to

          this.resultado = `${this.amount} ${fromName} equivalen a ${conversion} ${toName}`
        } else {
          Notify.create({
            type: 'negative',
            message: 'No se pudo obtener la tasa de conversión para las monedas seleccionadas.',
          })
        }
      } catch (error) {
        console.error('Error en la conversión:', error)
        Notify.create({
          type: 'negative',
          message: 'Error en la conversión. Verifica las monedas o el monto.',
        })
      }
    },
    intercambiar() {
      
      ;[this.from, this.to] = [this.to, this.from]
    },
  },
}
</script>

<style scoped>

.q-card {
  border-radius: 12px; 
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1); 
}


.q-select .q-field__selected-item {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}
</style>
