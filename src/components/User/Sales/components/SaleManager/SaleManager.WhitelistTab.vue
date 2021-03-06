<template>
  <div class="sale-manager-whitelist">
    <template v-if="isLoaded && whitelistedUsers.length">
      <table class="sale-manager-whitelist__table">
        <thead>
          <tr>
            <th>Participant</th>
            <th>Registered</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, index) in whitelistedUsers" :key="index">
            <td>{{ item.email }}</td>
            <td>{{ item.accounts ? 'Yes' : 'No' }}</td>
          </tr>
        </tbody>
      </table>
    </template>

    <template v-else-if="isLoaded && !whitelistedUsers.length">
      <p class="text">
        No whitelisted users yet.
      </p>
    </template>

    <template v-else-if="isLoadFailed">
      <p class="text danger">
        An error occurred. Please try again later.
      </p>
    </template>

    <template v-else>
      <p>
        Loading...
      </p>
    </template>
  </div>
</template>

<script>
import safeGet from 'lodash/get'

import { api } from '@/api'
import { ErrorHandler } from '@/utils/ErrorHandler'

export default {
  props: {
    sale: { type: Object, required: true },
  },

  data () {
    return {
      whitelistedUsers: [],
      isLoaded: false,
      isLoadFailed: false,
    }
  },

  async created () {
    await this.loadWhitelistedUsers()
  },

  methods: {
    safeGet,

    async loadWhitelistedUsers () {
      try {
        const { data } = await api.getWithSignature('/invites', {
          filter: {
            'sale-id': this.sale.id,
          },
        })
        this.whitelistedUsers = data
        this.isLoaded = true
      } catch (e) {
        this.isLoadFailed = true
        ErrorHandler.processWithoutFeedback(e)
      }
    },
  },
}
</script>

<style scoped lang="scss">
$side-padding: 0.8rem;

.sale-manager-whitelist__table {
  width: calc(100% + #{$side-padding} * 2);
  margin: 0 -$side-padding 0 -$side-padding;
  font-size: 1.6rem;
  border-collapse: collapse;

  & > thead > tr > th,
  & > tbody > tr > td {
    &:nth-child(1),
    &:nth-child(2) {
      text-align: left;
    }
    &:nth-child(3),
    &:nth-child(4) {
      text-align: right;
    }

    padding: 0.5rem 0;

    &:first-child {
      padding-left: $side-padding;
    }

    &:last-child {
      padding-right: $side-padding;
    }
  }

  & > tbody > tr:hover {
    background-color: rgba(0, 0, 0, 0.08);
  }
}
</style>
