<template>
  <div class="user-list">
    <div class="user-list__filters-wrp">
      <div class="app-list-filters">
        <select-field
          class="app-list-filters__field"
          label="Role"
          v-model="filters.role"
        >
          <option :value="''" />
          <option :value="kvAccountRoles.unverified">
            Unverified
          </option>
          <option :value="kvAccountRoles.general">
            General
          </option>
          <option :value="kvAccountRoles.corporate">
            Сorporate
          </option>
        </select-field>

        <input-field
          class="app-list-filters__field"
          v-model.trim="filters.requestor"
          label="Requestor"
          autocomplete-type="email"
        />
      </div>
    </div>

    <div class="user-list__list-wrp">
      <div class="app-list">
        <template v-if="list && list.length">
          <div class="app-list__header">
            <span class="app-list__cell user-list__email-cell">
              Email
            </span>
            <span class="app-list__cell app-list__cell--right">
              Account ID
            </span>
            <span class="app-list__cell app-list__cell--right">
              Role
            </span>
            <span class="app-list__cell app-list__cell--right">
              State
            </span>
          </div>

          <button
            class="app-list__li"
            v-for="item in list"
            :key="item.id"
            @click="showUserDetails(item.address)"
          >
            <span
              class="app-list__cell
                        app-list__cell--important
                        user-list__email-cell"
              :title="item.email"
            >
              {{ item.email }}
            </span>

            <span
              class="app-list__cell app-list__cell--right"
              :title="item.address"
            >
              {{ item.address | cropAddress }}
            </span>

            <span
              class="app-list__cell app-list__cell--right"
              :title="item.role | roleIdToString"
            >
              {{ item.role | roleIdToString }}
            </span>

            <account-state-getter
              class="app-list__cell app-list__cell--right"
              :account-id="item.address"
            />
          </button>
        </template>

        <template v-else>
          <div class="app-list__li-like">
            <template v-if="isLoading">
              <p>
                Loading...
              </p>
            </template>

            <template v-else>
              <p>
                Nothing here yet
              </p>
            </template>
          </div>
        </template>
      </div>

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
import { clearObject } from '@/utils/clearObject'

import SelectField from '@comcom/fields/SelectField'
import InputField from '@comcom/fields/InputField'

import { AccountStateGetter } from '@comcom/getters'
import { CollectionLoader } from '@/components/common'

import _ from 'lodash'

import { api } from '@/api'
import apiHelper from '@/apiHelper'

import { ErrorHandler } from '@/utils/ErrorHandler'
import { base } from '@tokend/js-sdk'
import { mapGetters } from 'vuex'

export default {
  components: {
    SelectField,
    InputField,
    AccountStateGetter,
    CollectionLoader,
  },

  data () {
    return {
      filters: {
        role: '',
        requestor: '',
      },
      list: [],
      isLoading: false,
    }
  },

  computed: {
    ...mapGetters([
      'kvAccountRoles',
    ]),
  },

  watch: {
    'filters.role' () {
      this.reloadCollectionLoader()
    },

    'filters.requestor': _.throttle(function () {
      this.reloadCollectionLoader()
    }, 1000),
  },

  methods: {
    async getList () {
      this.isLoading = true
      let response = {}
      try {
        const requestor =
          await this.getRequestorAccountId(this.filters.requestor)
        response = await api.getWithSignature('/identities', {
          filter: clearObject({
            role: this.filters.role,
            address: requestor,
          }),
        })
      } catch (error) {
        ErrorHandler.processWithoutFeedback(error)
      }
      this.isLoading = false
      return response
    },

    async getRequestorAccountId (requestor) {
      if (base.Keypair.isValidPublicKey(requestor)) {
        return requestor
      } else {
        try {
          const address = await apiHelper.users.getAccountIdByEmail(requestor)
          return address || requestor
        } catch (error) {
          return requestor
        }
      }
    },

    setList (data) {
      this.list = data
      this.isLoaded = true
    },
    extendList (data) {
      this.list = this.list.concat(data)
    },

    showUserDetails (id) {
      if (id) {
        this.$router.push({
          name: 'users.show',
          params: {
            id: id,
          },
        })
      }
    },

    reloadCollectionLoader () {
      this.$refs.collectionLoaderBtn.loadFirstPage()
    },
  },
}
</script>

<style lang="scss" scoped>
.user-list__filters-wrp {
  margin-bottom: 4rem;
}
</style>
