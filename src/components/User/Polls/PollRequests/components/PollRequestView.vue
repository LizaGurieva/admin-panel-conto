<template>
  <div class="poll-request-view">
    <div class="app__block">
      <div class="poll-request-view__request-attributes">
        <label class="data-caption">
          Request details
        </label>

        <ul class="key-value-list">
          <li>
            <span>Request ID</span>
            <span>{{ request.id }}</span>
          </li>
          <li>
            <span>Requestor</span>
            <email-getter
              :account-id="request.requestor.id"
              is-titled
            />
          </li>
          <li>
            <span>Request state</span>
            <request-state-formatter
              :state="request.state"
              is-colored
            />
          </li>

          <li>
            <span>Requested at</span>
            <date-formatter
              :date="request.createdAt"
              format="DD MMM YYYY HH:mm:ss"
            />
          </li>
        </ul>

        <template v-if="request.rejectReason">
          <label class="data-caption danger">
            Reject reason
          </label>
          <p class="text">
            {{ request.rejectReason }}
          </p>
        </template>

        <label class="data-caption">
          Poll question and answers
        </label>

        <p class="poll-request-view__question">
          {{ request.requestDetails.creatorDetails.question }}
        </p>

        <ul class="key-value-list">
          <li
            v-for="item in request.requestDetails.creatorDetails.choices"
            :key="item.number"
          >
            <span>Answer #{{ item.number }}</span>
            <span>
              {{ item.description }}
            </span>
          </li>
        </ul>

        <label class="data-caption">
          Poll details
        </label>

        <ul class="key-value-list">
          <li>
            <span>Number of choices</span>
            <span>{{ request.requestDetails.numberOfChoices }}</span>
          </li>

          <li>
            <span>Start time</span>
            <date-formatter
              :date="request.requestDetails.startTime"
              format="DD MMM YYYY HH:mm:ss"
            />
          </li>

          <li>
            <span>End time</span>
            <date-formatter
              :date="request.requestDetails.endTime"
              format="DD MMM YYYY HH:mm:ss"
            />
          </li>

          <li>
            <span>Permission type</span>
            <span>
              {{ request.requestDetails.permissionType | pollTypeToString }}
            </span>
          </li>

          <li>
            <span>Result provider</span>
            <email-getter
              :account-id="request.requestDetails.resultProvider.id"
              is-titled
            />
          </li>

          <li>
            <span>Vote confirmation required</span>
            <span>
              <template v-if="request.requestDetails.voteConfirmationRequired">
                Yes
              </template>
              <template v-else>
                No
              </template>
            </span>
          </li>
        </ul>
      </div>

      <template v-if="isRequestLoaded">
        <div
          class="poll-request-view__actions app__form-actions"
          v-if="request.stateI === REQUEST_STATES.pending"
        >
          <button
            class="app__btn"
            @click="approve"
            :disabled="isSubmitting"
          >
            Approve
          </button>
          <button
            class="app__btn app__btn--danger"
            :disabled="isSubmitting"
            @click="showRejectForm"
          >
            Reject
          </button>
        </div>
      </template>

      <template v-else>
        <p>
          <template v-if="isRequestFailed">
            An error occurred
          </template>
          <template v-else>
            Loading...
          </template>
        </p>
      </template>
    </div>

    <modal
      v-if="rejectForm.isShown"
      @close-request="hideRejectForm"
      max-width="40rem"
    >
      <p class="text">
        Reject reason
      </p>

      <form
        class="poll-request-view__reject-form"
        id="sale-reject-form"
        @submit.prevent="reject"
        novalidate
      >
        <div class="app__form-row">
          <text-field
            v-model="rejectForm.reason"
            :label="null"
            :disabled="formMixin.isDisabled"
            @blur="touchField('rejectForm.reason')"
            :error-message="getFieldErrorMessage(
              'rejectForm.reason',
              { maxLength: REJECT_REASON_MAX_LENGTH }
            )"
          />
        </div>

        <div class="app__form-row">
          <tick-field
            v-model="rejectForm.isPermanentReject"
            label="Reject permanently"
            disabled
          />
        </div>
      </form>

      <div class="poll-request-view__reject-form-actions app__form-actions">
        <button
          class="app__btn app__btn--danger"
          form="sale-reject-form"
          :disabled="formMixin.isDisabled"
        >
          Reject
        </button>
        <button
          class="app__btn-secondary"
          @click="hideRejectForm"
          :disabled="formMixin.isDisabled"
        >
          Cancel
        </button>
      </div>
    </modal>
  </div>
</template>

<script>
import FormMixin from '@/mixins/form.mixin'
import { required, maxLength } from '@/validators'

import { EmailGetter } from '@comcom/getters'
import {
  DateFormatter,
  RequestStateFormatter,
} from '@comcom/formatters'

import Modal from '@comcom/modals/Modal'
import { confirmAction } from '@/js/modals/confirmation_message'

import { REQUEST_STATES } from '@/constants'

import { ErrorHandler } from '@/utils/ErrorHandler'
import { api } from '@/api'
import apiHelper from '@/apiHelper'
import { Bus } from '@/utils/bus'

const REJECT_REASON_MAX_LENGTH = 255

export default {
  components: {
    DateFormatter,
    RequestStateFormatter,
    EmailGetter,
    Modal,
  },

  mixins: [FormMixin],

  props: {
    id: { type: String, required: true },
  },

  data () {
    return {
      request: {},
      rejectForm: {
        reason: '',
        isShown: false,
        isPermanentReject: true, // cuz poll can be rejected only permanently
      },
      isSubmitting: false,
      isRequestFailed: false,
      REQUEST_STATES,
      REJECT_REASON_MAX_LENGTH,
    }
  },

  validations () {
    return {
      rejectForm: {
        reason: {
          required,
          maxLength: maxLength(REJECT_REASON_MAX_LENGTH),
        },
      },
    }
  },

  computed: {
    isRequestLoaded () {
      return Object.keys(this.request).length > 0
    },
  },

  created () {
    this.getRequest(this.id)
  },

  methods: {
    async getRequest (id) {
      try {
        const { data } = await api.getWithSignature(
          `/v3/create_poll_requests/${this.id}`,
          { include: ['request_details'] }
        )
        this.request = data
      } catch (error) {
        ErrorHandler.processWithoutFeedback(error)
        this.isRequestFailed = true
      }
    },

    showRejectForm () {
      this.rejectForm.isShown = true
    },

    hideRejectForm () {
      this.rejectForm.isShown = false
    },

    async approve () {
      this.isSubmitting = true
      if (await confirmAction()) {
        try {
          await apiHelper.requests.approve(this.request)
          Bus.success('Sale request approved.')
          this.$router.push({ name: 'polls.requests' })
        } catch (error) {
          ErrorHandler.process(error)
        }
      }
      this.isSubmitting = false
    },

    async reject () {
      if (!this.isFormValid()) return

      this.hideRejectForm()
      this.isSubmitting = true
      try {
        await apiHelper.requests.reject(
          {
            reason: this.rejectForm.reason,
            isPermanent: this.rejectForm.isPermanentReject,
          },
          this.request
        )
        Bus.success('Sale request rejected successfully.')
        this.$router.push({ name: 'polls.requests' })
      } catch (error) {
        ErrorHandler.process(error)
      }
      this.isSubmitting = false
    },
  },
}
</script>

<style scoped lang="scss">
.poll-request-view__actions.app__form-actions {
  margin-top: 4rem;
  max-width: 30rem;
}

.poll-request-view__reject-form {
  margin-bottom: 2rem;
}

.poll-request-view__question {
  margin-bottom: 0.5rem;
  font-size: 1.6rem;
  line-height: 1.5;
}
</style>
