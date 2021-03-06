<template>
  <div class="price-chart-fetcher">
    <template v-if="priceHistory && Object.keys(priceHistory).length">
      <div class="app__block">
        <div class="price-chart-fetcher__filters-wrp">
          <scale-picker v-model="scale" />
        </div>

        <price-chart
          :price-history="priceHistory"
          :scale="scale"
          :asset-pair="filters.pair" />
      </div>
    </template>

    <template v-else-if="isLoaded">
      <div class="app__block price-chart__empty">
        <p>No chart data yet</p>
      </div>
    </template>

    <template v-else-if="isFailed">
      <div class="app__block price-chart__empty">
        <p>
          An error occurred while fetching chart data.
          Please try again later
        </p>
      </div>
    </template>

    <template v-else>
      <div class="app__block price-chart__empty">
        <p>Fetching chart data...</p>
      </div>
    </template>
  </div>
</template>

<script>
import { api } from '@/api'
import { AssetPair } from '../../models/AssetPair'

import PriceChart from './PriceChart.Renderer'
import ScalePicker from './PriceChart.ScalePicker'

import { ErrorHandler } from '@/utils/ErrorHandler'

export default {
  components: {
    PriceChart,
    ScalePicker,
  },

  props: {
    filters: { type: Object, required: true },
  },

  data () {
    return {
      priceHistory: {},
      scale: 'day',
      isLoaded: [],
      isFailed: [],
    }
  },

  watch: {
    'filters.pair' () { this.getPriceHistory() },
  },

  created () {
    this.getPriceHistory()
  },

  methods: {
    async getPriceHistory () {
      try {
        this.isLoaded = false
        this.isFailed = false
        this.priceHistory = {}
        const pair = new AssetPair(this.filters.pair)
        const endpoint = `/charts/${pair.base}-${pair.quote}`
        const { _rawResponse: response } = await api.getWithSignature(endpoint)
        this.priceHistory = response.data
        this.isLoaded = true
      } catch (error) {
        ErrorHandler.processWithoutFeedback(error)

        if (error.status === 404) {
          this.isLoaded = true
        } else {
          this.isFailed = true
        }
      }
    },
  },
}
</script>

<style scoped>
.price-chart-fetcher__filters-wrp {
  margin-bottom: 2rem;
}

.price-chart__empty.price-chart__empty {
  height: 30rem;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}
</style>
