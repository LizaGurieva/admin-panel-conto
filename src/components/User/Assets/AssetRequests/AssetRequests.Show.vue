<template>
  <div class="asset-requests-show app__block">
    <template v-if="isInitializing">
      <p class="text">
        Loading...
      </p>
    </template>

    <template v-else-if="isInitFailed">
      <p class="text danger">
        An error occurred. Please try again later
      </p>
    </template>

    <div
      class="asset-requests-show"
      v-else-if="assetRequest.id && !isRejecting"
    >
      <h2>
        <template v-if="assetRequest.type === ASSET_REQUEST_TYPES.createAsset">
          Asset creation request
        </template>

        <template
          v-else-if="assetRequest.type === ASSET_REQUEST_TYPES.updateAsset"
        >
          Asset update request
        </template>

        <template v-else>
          Asset request
        </template>
      </h2>
      <div class="asset-requests-show__row">
        <span class="asset-requests-show__key">
          Asset logo
        </span>
        <template
          v-if="safeGet(
            assetRequest, 'operationDetails.creatorDetails.logo.key'
          )"
        >
          <img-getter
            class="asset-requests-show__asset-logo"
            :file-key="assetRequest.operationDetails.creatorDetails.logo.key"
            alt="Asset logo"
          />
        </template>
        <template v-else>
          &mdash;
        </template>
      </div>
      <div class="asset-requests-show__row">
        <span class="asset-requests-show__key">
          Asset name
        </span>
        <span
          class="asset-requests-show__value"
          :title="assetRequest.operationDetails.creatorDetails.name"
        >
          {{ assetRequest.operationDetails.creatorDetails.name || '—' }}
        </span>
      </div>

      <div class="asset-requests-show__row">
        <span class="asset-requests-show__key">
          Asset code
        </span>
        <span class="asset-requests-show__value">
          {{ assetRequest.code || '—' }}
        </span>
      </div>

      <div
        class="asset-requests-show__row"
        v-if="assetRequest.type !== ASSET_REQUEST_TYPES.updateAsset"
      >
        <span class="asset-requests-show__key">
          Max issuance amount
        </span>
        <span class="asset-requests-show__value">
          {{
            assetRequest.maxAmount
              ? localizeAmount(assetRequest.maxAmount)
              : '—'
          }}
        </span>
      </div>

      <div
        class="asset-requests-show__row"
        v-if="assetRequest.type !== ASSET_REQUEST_TYPES.updateAsset"
      >
        <span class="asset-requests-show__key">
          Issued amount
        </span>
        <span class="asset-requests-show__value">
          {{ assetRequest.issuedAmount }}
        </span>
      </div>

      <div
        v-if="assetRequest.operationDetails.creatorDetails.expiresAt"
        class="asset-requests-show__row"
      >
        <span class="asset-requests-show__key">
          Expiration date
        </span>
        <span class="asset-requests-show__value">
          <date-formatter
            :date="getExpirationDate"
            format="DD MMM YYYY [at] HH:mm"
          />
        </span>
      </div>

      <div
        class="asset-requests-show__row"
        v-if="assetRequest.type !== ASSET_REQUEST_TYPES.updateAsset"
      >
        <span class="asset-requests-show__key">
          Preissuance signer
        </span>
        <email-getter
          v-if="assetRequest.signer"
          class="asset-requests-show__value"
          :account-id="assetRequest.signer"
          is-titled
        />
        <span
          v-else
          class="asset-requests-show__value"
          :title="assetRequest.signer"
        >
          —
        </span>
      </div>

      <div
        class="asset-requests-show__row"
        v-if="assetRequest.type !== ASSET_REQUEST_TYPES.updateAsset"
      >
        <span class="asset-requests-show__key">
          Type
        </span>
        <span class="asset-requests-show__value">
          {{ assetRequest.assetType | assetTypeToString }}
        </span>
      </div>

      <template v-if="assetRequest.policies">
        <div class="asset-requests-show__row asset-requests-show__row--policy">
          <span class="asset-requests-show__key">
            Policies
          </span>
          <div class="asset-requests-show__policies-wrapper">
            <template v-for="(policy, key) in ASSET_POLICIES_VERBOSE">
              <!-- eslint-disable max-len -->
              <span
                :key="key"
                class="asset-requests-show__key asset-requests-show__key--informative"
                v-if="assetRequest.policies & key"
              >
                {{ policy }}
              </span>
              <!-- eslint-enable max-len -->
            </template>
          </div>
        </div>
      </template>

      <div class="asset-requests-show__row">
        <span class="asset-requests-show__key">
          Terms
        </span>
        <span class="asset-requests-show__value">
          <template
            v-if="safeGet(
              assetRequest, 'operationDetails.creatorDetails.terms.key'
            )"
            :mime-type="safeGet(
              assetRequest,
              'operationDetails.details.terms.mimeType'
            )"
          >
            <user-doc-link-getter
              :file-key="assetRequest.operationDetails.creatorDetails.terms.key"
            >
              Open file
            </user-doc-link-getter>
          </template>
          <template v-else>
            &mdash;
          </template>
        </span>
      </div>

      <template v-if="assetRequest.stellarAssetCode">
        <div
          class="asset-requests-show__row"
        >
          <span class="asset-requests-show__key">
            Stellar asset code
          </span>
          <span class="asset-requests-show__value">
            {{ assetRequest.stellarAssetCode }}
          </span>
        </div>

        <div
          class="asset-requests-show__row"
        >
          <span class="asset-requests-show__key">
            Stellar asset type
          </span>
          <span class="asset-requests-show__value">
            {{ stellarAssetType }}
          </span>
        </div>

        <div
          class="asset-requests-show__row"
        >
          <span class="asset-requests-show__key">
            Stellar withdraw
          </span>
          <span class="asset-requests-show__value">
            {{ assetRequest.stellarWithdraw ? 'Yes' : 'No' }}
          </span>
        </div>

        <div
          class="asset-requests-show__row"
        >
          <span class="asset-requests-show__key">
            Stellar deposit
          </span>
          <span class="asset-requests-show__value">
            {{ assetRequest.stellarDeposit ? 'Yes' : 'No' }}
          </span>
        </div>
      </template>

      <div class="asset-requests-show__row">
        <span class="asset-requests-show__key">
          Creation date
        </span>
        <date-formatter
          :date="assetRequest.creationDate"
          format="DD MMM YYYY HH:mm:ss"
          class="asset-requests-show__value"
        />
      </div>

      <div class="asset-requests-show__row">
        <span class="asset-requests-show__key">
          Update date
        </span>
        <date-formatter
          :date="assetRequest.updateDate"
          format="DD MMM YYYY HH:mm:ss"
          class="asset-requests-show__value"
        />
      </div>

      <!-- eslint-disable-next-line max-len -->
      <template v-if="assetRequest.state !== CREATE_ASSET_REQUEST_STATES.pending.codeVerbose">
        <div class="asset-requests-show__row">
          <span class="asset-requests-show__key">
            State
          </span>

          <span class="asset-requests-show__value">
            {{ verbozify(assetRequest.state) }}
          </span>
        </div>
      </template>

      <!-- eslint-disable-next-line max-len -->
      <template v-if="assetRequest.state === CREATE_ASSET_REQUEST_STATES.rejected.codeVerbose">
        <div class="asset-requests-show__reject-reason-wrp">
          <text-field
            label="Reject reason"
            :value="assetRequest.rejectReason"
            :readonly="true"
          />
        </div>
      </template>

      <!-- eslint-disable max-len -->
      <div
        class="asset-requests-show__buttons"
        v-if="assetRequest.state === CREATE_ASSET_REQUEST_STATES.pending.codeVerbose"
      >
        <button
          class="app__btn"
          :disabled="isPending"
          @click="fulfill"
        >
          Fulfill
        </button>

        <button
          class="app__btn-secondary"
          :disabled="isPending"
          @click="isRejecting = true"
        >
          Reject
        </button>
      </div>
    </div>
    <!-- eslint-enable max-len -->

    <asset-request-reject-form
      @close="isRejecting = false"
      :asset-request="assetRequest"
      v-else-if="isRejecting"
    />
  </div>
</template>

<script>
import TextField from '@comcom/fields/TextField'
import AssetRequestRejectForm from './components/AssetRequestRejectForm'
import moment from 'moment'

import { ImgGetter, EmailGetter, UserDocLinkGetter } from '@comcom/getters'
import { DateFormatter } from '@comcom/formatters'

import { confirmAction } from '@/js/modals/confirmation_message'

import localize from '@/utils/localize'
import { verbozify } from '@/utils/verbozify'
import safeGet from 'lodash/get'

import { AssetRequest } from '@/apiHelper/responseHandlers/requests/AssetRequest'
import { ErrorHandler } from '@/utils/ErrorHandler'
import { Bus } from '@/utils/bus'

import {
  ASSET_POLICIES_VERBOSE,
  CREATE_ASSET_REQUEST_STATES,
  ASSET_REQUEST_TYPES,
} from '@/constants'

import { api } from '@/api'

const STELLAR_TYPES = {
  creditAlphanum4: 'credit_alphanum4',
  creditAlphanum12: 'credit_alphanum12',
  native: 'native',
}

// TODO: extract to AssetRequestForm
export default {
  components: {
    AssetRequestRejectForm,
    TextField,
    ImgGetter,
    EmailGetter,
    DateFormatter,
    UserDocLinkGetter,
  },

  props: {
    id: { type: String, required: true },
  },

  data () {
    return {
      assetRequest: {},
      isRejecting: false,
      isPending: false,
      isInitializing: false,
      isInitFailed: false,
      ASSET_POLICIES_VERBOSE,
      ASSET_REQUEST_TYPES,
      CREATE_ASSET_REQUEST_STATES,
    }
  },

  computed: {
    isCancellable () {
      const isPending = this.assetRequest.state ===
        CREATE_ASSET_REQUEST_STATES.pending.codeVerbose
      const isCancellableRequestor =
        this.assetRequest.requestor === this.$store.getters.masterId
      return isPending && isCancellableRequestor
    },
    stellarAssetType () {
      let label

      switch (this.assetRequest.stellarAssetType) {
        case STELLAR_TYPES.creditAlphanum4:
          label = 'Alphanumeric 4'
          break

        case STELLAR_TYPES.creditAlphanum12:
          label = 'Alphanumeric 12'
          break

        case STELLAR_TYPES.native:
          label = 'Native'
          break

        default:
          label = '[UNKNOWN_STELLAR_ASSET_TYPE]'
          break
      }

      return label
    },

    getExpirationDate () {
      // eslint-disable-next-line max-len
      return moment.unix(this.assetRequest.operationDetails.creatorDetails.expiresAt)
    },
  },

  async created () {
    this.isInitializing = true

    try {
      const { data } = await api.getWithSignature(`/v3/requests/${this.id}`, {
        include: ['request_details'],
      })
      this.assetRequest = new AssetRequest(data)
    } catch (error) {
      ErrorHandler.processWithoutFeedback(error)
      this.isInitFailed = true
    }

    this.isInitializing = false
  },

  methods: {
    verbozify,
    safeGet,

    async fulfill () {
      this.isPending = true
      if (await confirmAction()) {
        try {
          await this.assetRequest.fulfill()
          Bus.success('Asset successfully created')
          this.$router.push({ name: 'assets' })
        } catch (error) {
          ErrorHandler.process(error)
        }
      }
      this.isPending = false
    },

    localizeAmount: localize,
  },
}
</script>

<style lang="scss" scoped>
@import "../../../../assets/scss/colors";

.asset-requests-show {
  max-width: 64rem;
}

.asset-requests-show__row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 1rem;
}

.asset-requests-show__key {
  &--informative {
    color: $color-info;
    text-align: right;

    &:before {
      content: "\2713";
    }
  }
}

.asset-requests-show__value {
  max-width: 24rem;
  overflow: hidden;
  text-overflow: ellipsis;
  text-align: right;
}

.asset-requests-show__buttons {
  display: flex;
  margin-top: 4.5rem;

  & > button {
    flex: 0.33;
  }

  button:first-child {
    margin-right: 1rem;
  }
}

.asset-requests-show__reject-reason-wrp {
  margin-top: 2rem;
}

.asset-requests-show__asset-logo {
  max-width: 16rem;
  max-height: 16rem;
}

.asset-requests-show__policies-wrapper {
  display: flex;
  flex-direction: column;
}
</style>
