<template>
  <div class="sale-list">
    <h2>
      Sale list
    </h2>

    <div class="sale-list__filters-wrp">
      <div class="app-list-filters sale-list__filters">
        <input-field
          class="app-list-filters__field sale-list__field"
          label="Asset code"
          v-model="filters.baseAsset"
        />
        <input-field
          class="app-list-filters__field sale-list__field"
          label="Owner"
          placeholder="Address (full match)"
          v-model="owner"
          autocomplete-type="email"
        />
        <select-field
          class="app-list-filters__field sale-list__field"
          label="State"
          v-model="filters.state">
          <option :value="SALE_STATES.open">
            Open
          </option>
          <option :value="SALE_STATES.closed">
            Closed
          </option>
          <option :value="SALE_STATES.cancelled">
            Cancelled
          </option>
        </select-field>
        <input-date-field
          label="Start date"
          class="sale-list__field sale-list__field-margin-top"
          :enable-time="false"
          v-model="filters.startDate"
        />
        <!-- eslint-disable max-len -->
        <input-date-field
          label="End date"
          class="sale-list__field sale-list__field-margin-left sale-list__field-margin-top"
          :enable-time="false"
          v-model="filters.endDate"
        />
        <select-field
          class="sale-list__field sale-list__field-margin-left sale-list__field-margin-top"
          label="Type"
          v-model="filters.type">
          <option :value="''" />
          <option :value="SALE_TYPES.basicSale">
            Basic
          </option>
          <option :value="SALE_TYPES.crowdFunding">
            Crowdfunding
          </option>
          <option :value="SALE_TYPES.fixedPrice">
            Fixed price
          </option>
          <option :value="SALE_TYPES.immediate">
            Immediate
          </option>
        </select-field>
        <!-- eslint-enable max-len -->
      </div>
    </div>

    <div class="sale-list__list-wrp">
      <template v-if="list && list.length">
        <ul class="app-list">
          <div class="app-list__header">
            <span class="app-list__cell">
              <!-- empty -->
            </span>
            <span class="app-list__cell">
              Name
            </span>
            <span class="app-list__cell">
              State
            </span>
            <span class="app-list__cell">
              Owner
            </span>
          </div>

          <router-link
            class="app-list__li"
            v-for="item in list"
            :key="item.id"
            :to="{ name: 'sales.show', params: { id: item.id }}"
          >
            <span
              class="app-list__cell app-list__cell--important"
              :title="item.baseAsset.id"
            >
              {{ item.baseAsset.id }}
            </span>
            <span
              class="app-list__cell"
              :title="item.details.name"
            >
              {{ item.details.name }}
            </span>
            <span class="app-list__cell">
              <template v-if="item.saleState.value === SALE_STATES.open">
                Open
              </template>
              <template v-else-if="item.saleState.value === SALE_STATES.closed">
                Closed
              </template>
              <template
                v-else-if="item.saleState.value === SALE_STATES.cancelled"
              >
                Cancelled
              </template>
            </span>
            <span class="app-list__cell">
              <email-getter
                :account-id="item.owner.id"
                is-titled
              />
            </span>
          </router-link>
        </ul>
      </template>

      <template v-else>
        <ul class="app-list">
          <li class="app-list__li-like">
            <template v-if="isLoaded">
              Nothing here yet
            </template>
            <template v-else>
              Loading...
            </template>
          </li>
        </ul>
      </template>
      <div class="app__more-btn-wrp">
        <collection-loader
          :first-page-loader="getList"
          @first-page-load="setList"
          @next-page-load="extendList"
          ref="collectionLoaderBtn"
        />
      </div>
    </div>
  </div>
</template>

<script>
import {
  InputField,
  InputDateField,
  SelectField,
} from '@comcom/fields'
import { EmailGetter } from '@comcom/getters'
import { CollectionLoader } from '@/components/common'

import { api } from '@/api'
import apiHelper from '@/apiHelper'

import { SALE_STATES } from '@/constants'
import { SALE_TYPES } from '@tokend/js-sdk'

import _ from 'lodash'

import config from '@/config'
import { ErrorHandler } from '@/utils/ErrorHandler'

export default {
  components: {
    InputField,
    SelectField,
    EmailGetter,
    InputDateField,
    CollectionLoader,
  },

  data () {
    return {
      SALE_STATES,
      SALE_TYPES,
      list: [],
      rawList: [],
      isLoaded: false,
      owner: '',
      filters: {
        baseAsset: '',
        owner: '',
        state: SALE_STATES.open,
        startDate: '',
        endDate: '',
        type: '',
      },
    }
  },

  watch: {
    'filters.state' () {
      this.reloadCollectionLoader()
    },

    'filters.type' () {
      this.reloadCollectionLoader()
    },

    'owner': _.throttle(async function () {
      this.filters.owner = await this.getOwner()
      this.reloadCollectionLoader()
    }, 1000),

    'filters.baseAsset': _.throttle(function () {
      this.reloadCollectionLoader()
    }, 1000),

    'filters.startDate' () {
      this.reloadCollectionLoader()
    },

    'filters.endDate' () {
      this.reloadCollectionLoader()
    },
  },

  methods: {
    async getOwner () {
      let owner
      const emailRegExp = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
      if (emailRegExp.test(this.owner)) {
        owner = await apiHelper.users.getAccountIdByEmail(this.owner)
      }

      return owner || this.owner
    },

    async getList () {
      this.isLoaded = false
      let response = {}
      try {
        response = await api.getWithSignature('/v3/sales', {
          filter: {
            base_asset: this.filters.baseAsset,
            owner: this.filters.owner,
            state: this.filters.state,
            min_start_time: this.filters.startDate,
            max_end_time: this.filters.endDate,
            sale_type: this.filters.type,
          },
          page: {
            limit: this.filters.limit || config.PAGE_LIMIT,
            order: this.filters.order || 'desc',
          },
          include: ['base_asset', 'quote_assets', 'default_quote_asset'],
        })
      } catch (error) {
        ErrorHandler.processWithoutFeedback(error)
      }
      this.isLoaded = true
      return response
    },

    setList (data) {
      this.list = data
    },

    async extendList (data) {
      this.list = this.list.concat(data)
    },

    reloadCollectionLoader () {
      this.$refs.collectionLoaderBtn.loadFirstPage()
    },
  },
}
</script>

<style scoped lang="scss">
.sale-list__filters-wrp {
  margin-bottom: 4rem;
}

.sale-list__filters {
  flex-wrap: wrap;
}

.sale-list__field {
  min-width: calc(33.333333% - 2rem);
}

.sale-list__field-margin {
  &-left {
    margin-left: 2rem;
  }
  &-top {
    margin-top: 2rem;
  }
}
</style>
