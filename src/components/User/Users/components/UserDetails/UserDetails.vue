<template>
  <div class="user-details">
    <div class="app__block">
      <h2>User details</h2>

      <template v-if="isLoaded">
        <section
          class="user-details__section"
          v-if="requestToReview.state"
        >
          <ul class="key-value-list">
            <li>
              <h3>Current request state</h3>
              <p
                :class="`user-details__state-info
                         user-details__state-info--${requestToReview.state}`">
                {{ requestToReview.state }}
              </p>
            </li>
            <li>
              <h3>Role to set</h3>
              <p class="user-details__role-info">
                {{
                  roleTypeVerbose[requestToReview.accountRoleToSet
                    || verifiedRequest.accountRoleToSet]
                }}
              </p>
            </li>
          </ul>
          <p v-if="requestToReview.isRejected">
            Reason: {{ requestToReview.rejectReason }}
          </p>

          <template v-if="requestToReview.externalDetails">
            <div class="user-details__ext-details-wrp">
              <h3>External details (provided by external services)</h3>
              <external-details-viewer
                :external-details="requestToReview.externalDetails"
              />
            </div>
          </template>
        </section>

        <section class="user-details__section">
          <account-section
            :user="user"
            :original-role="userRole"
            :block-reason="latestBlockedRequest.blockReason"
          />
        </section>

        <template v-if="requestToReview.state">
          <section class="user-details__section">
            <template v-if="isKycLoaded">
              <general-kyc
                v-if="
                  requestToReview.accountRoleToSet === kvAccountRoles.general
                "
                :blob-id="requestToReview.blobId"
                :user="user" />
              <verified-kyc-viewer
                v-if="
                  requestToReview.accountRoleToSet === kvAccountRoles.usVerified
                "
                :kyc="kyc"
                :user="user" />
              <accredited-kyc-viewer
                v-if="
                  requestToReview.accountRoleToSet ===
                    kvAccountRoles.usAccredited
                "
                :kyc="kyc"
                :user="user" />
            </template>
            <template v-else-if="isKycLoadFailed">
              <p class="danger">
                An error occurred. Please try again later.
              </p>
            </template>
            <template v-else>
              <p>Loading...</p>
            </template>
            <kyc-syndicate-section
              v-if="
                requestToReview.accountRoleToSet === kvAccountRoles.corporate
              "
              :user="user"
              :blob-id="requestToReview.blobId"
            />
          </section>
          <!-- eslint-enable max-len -->
        </template>

        <template v-if="verifiedRequest.state">
          <!-- eslint-disable max-len -->
          <section
            v-if="verifiedRequest.accountRoleToSet !== kvAccountRoles.unverified && !isUserBlocked"
            class="user-details__section"
          >
            <template v-if="isKycLoaded">
              <h2>Previous approved KYC Request</h2>
              <general-kyc
                v-if="verifiedRequest.accountRoleToSet === kvAccountRoles.general"
                :blob-id="verifiedRequest.blobId"
                :user="user"
              />
              <verified-kyc-viewer
                v-if="verifiedRequest.accountRoleToSet === kvAccountRoles.usVerified"
                :blob-id="verifiedRequest.blobId"
                :user="user"
              />
              <accredited-kyc-viewer
                v-if="verifiedRequest.accountRoleToSet === kvAccountRoles.usAccredited"
                :kyc="kyc"
                :user="user"
                :blob-id="verifiedRequest.blobId"
              />
            </template>
            <template v-else-if="isKycLoadFailed">
              <p class="danger">
                An error occurred. Please try again later.
              </p>
            </template>
            <template v-else>
              <p>Loading...</p>
            </template>
            <kyc-syndicate-section
              v-if="verifiedRequest.accountRoleToSet === kvAccountRoles.corporate"
              :user="user"
              :blob-id="verifiedRequest.blobId"
            />
          </section>
          <!-- eslint-enable max-len -->

          <div
            v-if="requestToReview.state"
            class="user-details__latest-request"
          >
            <h3>Latest request</h3>
            <!-- eslint-disable max-len -->
            <p class="text">
              Create a "{{ requestToReview.accountRoleToSet | roleIdToString }}" account:
              {{ requestToReview.state }}
            </p>
            <!-- eslint-enable max-len -->
          </div>
        </template>

        <div class="user-details__actions-wrp">
          <template v-if="requestToReview.state">
            <request-actions
              class="user-details__actions"
              :user="user"
              :request-to-review="requestToReview"
              :latest-approved-request="verifiedRequest"
              @reviewed="getUpdatedUser"
            />
          </template>

          <template v-else>
            <block-actions
              class="user-details__actions"
              :user="user"
              :latest-approved-request="latestApprovedRequest"
              :verified-request="verifiedRequest"
              :is-pending.sync="isPending"
              @updated="getUpdatedUser"
            />

            <reset-actions
              v-if="!isUserBlocked"
              class="user-details__actions"
              :user="user"
              :verified-request="verifiedRequest"
              :is-pending.sync="isPending"
              @reset="getUpdatedUser"
            />
          </template>
        </div>
      </template>

      <template v-else-if="!isFailed">
        <p>Loading...</p>
      </template>

      <template v-else>
        <p class="danger">
          An error occurred. Please try again later.
        </p>
      </template>
    </div>
  </div>
</template>

<script>
import AccountSection from './UserDetails.Account'

import KycSyndicateSection from '@/components/User/Sales/components/SaleManager/SaleManager.SyndicateTab'

import GeneralKyc from './UserDetails.GeneralKyc'
import VerifiedKycViewer from './UserDetails.VerifiedKycViewer'
import AccreditedKycViewer from './UserDetails.AccreditedKycViewer'

import RequestActions from './UserDetails.Request'
import ResetActions from './UserDetails.Reset'
import BlockActions from './UserDetails.Block'

import ExternalDetailsViewer from './UserDetails.ExternalDetailsViewer'

import { api } from '@/api'
import { ErrorHandler } from '@/utils/ErrorHandler'

import { ChangeRoleRequest } from '@/apiHelper/responseHandlers/requests/ChangeRoleRequest'
import { fromKycTemplate } from '../../../../../utils/kyc-tempater'
import deepCamelCase from 'camelcase-keys-deep'

import { mapGetters } from 'vuex'

const OPERATION_TYPE = {
  createKycRequest: '22',
}
const EVENTS = {
  reviewed: 'reviewed',
}

export default {
  components: {
    AccountSection,
    ExternalDetailsViewer,
    KycSyndicateSection,
    RequestActions,
    ResetActions,
    BlockActions,
    AccreditedKycViewer,
    VerifiedKycViewer,
    GeneralKyc,
  },

  props: {
    id: { type: String, required: true },
  },

  data () {
    return {
      OPERATION_TYPE,
      isLoaded: false,
      isFailed: false,
      isPending: false,
      isKycLoaded: false,
      isKycLoadFailed: false,
      user: {},
      requests: [],
      verifiedRequest: {},
      kyc: {},
    }
  },

  computed: {
    ...mapGetters([
      'kvAccountRoles',
    ]),

    roleTypeVerbose () {
      return {
        [ this.kvAccountRoles.general ]: 'General',
        [ this.kvAccountRoles.usVerified ]: 'US Verified',
        [ this.kvAccountRoles.usAccredited ]: 'US Accredited',
        [ this.kvAccountRoles.corporate ]: 'Corporate',
        [ this.kvAccountRoles.notVerified ]: 'Not Verified',
      }
    },

    isUserBlocked () {
      return this.user.role === this.kvAccountRoles.blocked
    },

    latestApprovedRequest () {
      return (
        this.requests.find(item => item.isApproved)
      ) || new ChangeRoleRequest({})
    },

    requestToReview () {
      return (
        this.requests.find(item => item.isPending || item.isRejected)
      ) || new ChangeRoleRequest({})
    },

    latestBlockedRequest () {
      return this.requests.find(item => {
        return item.accountRoleToSet === this.kvAccountRoles.blocked
      }) || new ChangeRoleRequest({})
    },

    latestNonBlockedRequest () {
      return this.requests.find(item => {
        return item.isApproved &&
          item.accountRoleToSet !== this.kvAccountRoles.blocked
      }) || new ChangeRoleRequest({})
    },

    userRole () {
      return String(
        this.latestNonBlockedRequest.accountRoleToSet ||
        this.kvAccountRoles.unverified
      )
    },
  },

  async created () {
    await this.getUser()

    if (this.requestToReview.state) {
      await this.getKyc(this.requestToReview.blobId)
    } else if (this.verifiedRequest.state) {
      await this.getKyc(this.verifiedRequest.blobId)
    }
  },

  methods: {
    async getUser () {
      this.isLoaded = false
      this.isFailed = false
      try {
        const [user, requests] = await Promise.all([
          api.getWithSignature('/identities', {
            filter: { address: this.id },
          }),
          api.getWithSignature('/v3/change_role_requests', {
            page: { order: 'desc' },
            filter: { requestor: this.id },
            include: ['request_details'],
          }),
        ])
        this.user = user.data[0]
        this.requests = requests.data
          ? requests.data.map(item => new ChangeRoleRequest(item))
          : []
        await this.loadVerifiedRequest()
        this.isLoaded = true
      } catch (error) {
        ErrorHandler.processWithoutFeedback(error)
        this.isFailed = true
      }
    },

    async loadVerifiedRequest () {
      if (this.latestApprovedRequest) {
        const requestId = this.latestApprovedRequest.relatedRequestId

        if (requestId && requestId !== '0') {
          const endpoint = `/v3/change_role_requests/${requestId}`
          const { data } = await api.getWithSignature(endpoint, {
            include: ['request_details'],
          })
          this.verifiedRequest = new ChangeRoleRequest(data)
        } else if (!requestId) {
          this.verifiedRequest = this.latestApprovedRequest
        }
      }
    },

    async getUpdatedUser () {
      this.isLoaded = false
      this.isFailed = false

      setTimeout(async () => {
        await this.getUser()
        this.$emit(EVENTS.reviewed)
      }, 5000)
    },

    async getKyc (blobId) {
      if (!blobId) return

      this.isKycLoaded = false
      this.isKycLoadFailed = false

      try {
        const endpoint = `/accounts/${this.user.address}/blobs/${blobId}`
        const { data } = await api.getWithSignature(endpoint)
        const kycFormResponse = data
        this.kyc = deepCamelCase(
          fromKycTemplate(JSON.parse(kycFormResponse.value))
        )
        this.isKycLoaded = true
      } catch (error) {
        ErrorHandler.process(error)
        this.isKycLoadFailed = true
      }
    },
  },
}
</script>

<style lang="scss" scoped>
@import "~@/assets/scss/colors";

.user-details__section {
  flex: 1;

  & > h1, & > h2, & > h3 {
    margin-bottom: 1.2rem;
  }
}

.user-details__section:not(:first-of-type) {
  margin-top: 3rem;
}

.user-details__actions-wrp {
  display: flex;
  align-items: flex-end;
  margin-top: 4rem;
}

.user-details__latest-request {
  margin-top: 3.5rem;
}

.user-details__actions:not(:first-child) {
  margin-left: 1rem;
}

.user-details__heading {
  display: flex;
  line-height: 100%;
  align-items: center;

  span {
    margin-right: 1rem;
  }
}

.user-details__state-info {
  text-transform: capitalize;

  &--approved { color: $color-success; }
  &--pending { color: $color-active; }
  &--rejected { color: $color-danger; }
}

.user-details__ext-details-wrp {
  margin: 2rem 0;
}
</style>
