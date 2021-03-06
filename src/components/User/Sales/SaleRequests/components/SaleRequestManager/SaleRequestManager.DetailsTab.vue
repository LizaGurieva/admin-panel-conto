<template>
  <div class="sale-rm-details-tab">
    <div class="sale-rm-details-tab__row">
      <div class="sale-rm-details-tab__row-item">
        <label class="data-caption">
          Request details
        </label>
        <ul class="key-value-list">
          <li>
            <span>Requestor</span>
            <email-getter
              :account-id="request.sale.requestor.id"
              is-titled
            />
          </li>
          <li>
            <span>Request state</span>
            <request-state-formatter
              :state="request.sale.state"
              is-colored
            />
          </li>

          <li>
            <span>Requested at</span>
            <date-formatter
              :date="request.sale.createdAt"
              format="DD MMM YYYY HH:mm:ss"
            />
          </li>
        </ul>

        <template v-if="request.sale.rejectReason">
          <label class="data-caption danger">
            Reject reason
          </label>
          <p class="text">
            {{ request.sale.rejectReason }}
          </p>
        </template>
      </div>

      <div class="sale-rm-details-tab__row-item" />
    </div>

    <div class="sale-rm-details-tab__row">
      <div class="sale-rm-details-tab__row-item">
        <label class="data-caption">
          Asset details
        </label>
        <ul class="key-value-list">
          <li>
            <span>Name</span>
            <span>
              <template v-if="safeGet(request, 'asset.details.name')">
                {{ request.asset.details.name }}
              </template>
              <template v-else>
                &mdash;
              </template>
            </span>
          </li>
          <li>
            <span>Code</span>
            <span>{{ request.asset.id }}</span>
          </li>
          <li>
            <span>Initial preissued amount</span>
            <asset-amount-formatter :amount="request.asset.maxIssuanceAmount" />
          </li>
          <li>
            <span>Max issuance amount</span>
            <asset-amount-formatter :amount="request.asset.maxIssuanceAmount" />
          </li>
          <li>
            <span>Preissuance signer</span>
            <email-getter
              :account-id="request.asset.preIssuanceAssetSigner"
              is-titled
            />
          </li>
          <li>
            <span>Policies</span>
            <asset-policies-formatter
              :policy-mask="request.asset.policies.value"
            />
          </li>
          <li>
            <span>Terms</span>
            <span>
              <template v-if="safeGet(request, 'asset.details.terms.key')">
                <doc-link-getter :file-key="request.asset.details.terms.key">
                  Open file
                </doc-link-getter>
              </template>

              <template v-else>
                (No document)
              </template>
            </span>
          </li>
          <template v-if="safeGet(request, 'asset.details.stellar.assetCode')">
            <li>
              <span>Stellar asset code</span>
              <span>{{ request.asset.details.stellar.assetCode }}</span>
            </li>
            <li>
              <span>Stellar asset type</span>
              <span>{{ stellarAssetType }}</span>
            </li>
            <li>
              <span>Stellar withdraw</span>
              <span>
                {{ request.asset.details.stellar.withdraw ? 'Yes' : 'No' }}
              </span>
            </li>
            <li>
              <span>Stellar deposit</span>
              <span>
                {{ request.asset.details.stellar.deposit ? 'Yes' : 'No' }}
              </span>
            </li>
          </template>
        </ul>
      </div>

      <div class="sale-rm-details-tab__row-item">
        <label class="data-caption">
          Asset logo
        </label>
        <template v-if="safeGet(request, 'asset.details.logo.key')">
          <img-getter
            class="sale-rm-details-tab__asset-logo"
            :file-key="request.asset.details.logo.key"
            alt="Asset logo"
          />
        </template>
        <template v-else>
          <p>(No logo yet)</p>
        </template>
      </div>
    </div>

    <div class="sale-rm-details-tab__row">
      <div class="sale-rm-details-tab__row-item">
        <label class="data-caption">
          Sale details
        </label>
        <ul class="key-value-list">
          <li>
            <span>Name</span>
            <span>
              <template v-if="safeGet(saleDetails, 'creatorDetails.name')">
                {{ saleDetails.creatorDetails.name }}
              </template>
              <template v-else>
                (Not provided yet)
              </template>
            </span>
          </li>
          <li>
            <span>Type</span>
            <span>
              {{ LOCALIZED_SALE_TYPES[safeGet(saleDetails, 'saleType.value')] }}
            </span>
          </li>
          <li>
            <span>Whitelisted</span>
            <span>{{ isSaleWhitelisted ? 'Yes' : 'No' }}</span>
          </li>
          <li>
            <span>Start time</span>
            <date-formatter
              :date="saleDetails.startTime"
              format="DD MMM YYYY HH:mm:ss"
            />
          </li>
          <li>
            <span>End time</span>
            <date-formatter
              :date="saleDetails.endTime"
              format="DD MMM YYYY HH:mm:ss"
            />
          </li>
          <li>
            <span>Soft cap</span>
            <asset-amount-formatter
              :amount="saleDetails.softCap"
              :asset="saleDetails.defaultQuoteAsset.id"
            />
          </li>
          <li>
            <span>Hard cap</span>
            <asset-amount-formatter
              :amount="saleDetails.hardCap"
              :asset="saleDetails.defaultQuoteAsset.id"
            />
          </li>
          <li>
            <span>Max {{ saleDetails.baseAsset.id }} amount to be sold</span>
            <asset-amount-formatter
              :amount="saleDetails.baseAssetForHardCap"
              :asset="saleDetails.baseAsset.id"
            />
          </li>
        </ul>

        <label class="data-caption">
          Short description
        </label>
        <p
          class="text sale-rm-details-tab__short-description"
          :title="safeGet(saleDetails, 'creatorDetails.shortDescription')"
        >
          <template
            v-if="safeGet(saleDetails, 'creatorDetails.shortDescription')"
          >
            {{ saleDetails.creatorDetails.shortDescription }}
          </template>
          <template v-else>
            (Not provided yet)
          </template>
        </p>
      </div>

      <div class="sale-rm-details-tab__row-item">
        <label class="data-caption">
          Sale logo
        </label>
        <template v-if="safeGet(saleDetails, 'creatorDetails.logo.key')">
          <img-getter
            class="sale-rm-details-tab__sale-logo"
            :file-key="saleDetails.creatorDetails.logo.key"
            alt="Sale logo"
          />
        </template>
        <template v-else>
          <p>(No logo yet)</p>
        </template>
      </div>
    </div>
  </div>
</template>

<script>
import { EmailGetter, ImgGetter, DocLinkGetter } from '@comcom/getters'
import {
  AssetAmountFormatter,
  DateFormatter,
  AssetPoliciesFormatter,
  RequestStateFormatter,
} from '@comcom/formatters'

import { SALE_DEFINITION_TYPES, LOCALIZED_SALE_TYPES } from '@/constants'

import get from 'lodash/get'

const STELLAR_TYPES = {
  creditAlphanum4: 'credit_alphanum4',
  creditAlphanum12: 'credit_alphanum12',
  native: 'native',
}

export default {
  components: {
    EmailGetter,
    ImgGetter,
    DocLinkGetter,
    AssetAmountFormatter,
    DateFormatter,
    AssetPoliciesFormatter,
    RequestStateFormatter,
  },

  props: {
    request: { type: Object, required: true },
  },
  data () {
    return {
      LOCALIZED_SALE_TYPES,
    }
  },

  computed: {
    saleDetails () {
      return this.request.sale.requestDetails
    },

    isSaleWhitelisted () {
      return this.saleDetails.accessDefinitionType.value ===
        SALE_DEFINITION_TYPES.whitelist
    },

    stellarAssetType () {
      let label

      switch (this.request.asset.details.stellar.assetType) {
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
  },
  methods: {
    safeGet: get,
  },
}
</script>

<style scoped lang="scss">
.sale-rm-details-tab__row {
  display: flex;

  & + & {
    margin-top: 4rem;
  }
}

.sale-rm-details-tab__row-item {
  flex: 1;
  min-width: 0;

  & + & {
    margin-left: 4rem;
  }
}

.sale-rm-details-tab__asset-logo,
.sale-rm-details-tab__sale-logo {
  margin-top: 0.5rem;
}

.sale-rm-details-tab__asset-logo {
  max-width: 6.4rem;
  max-height: 6.4rem;
}

.sale-rm-details-tab__sale-logo {
  max-width: 20rem;
  max-height: 20rem;
}

.sale-rm-details-tab__short-description {
  overflow: hidden;
  text-overflow: ellipsis;
}
</style>
