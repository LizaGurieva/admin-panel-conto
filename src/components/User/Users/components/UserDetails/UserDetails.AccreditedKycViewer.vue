<template>
  <div class="accredited-kyc-viewer">
    <div class="accredited-kyc-viewer__warning-msg">
      <span class="accredited-kyc-viewer__header-title">
        Current user is applying for Accredited KYC
      </span>
      <span class="accredited-kyc-viewer__header-subtitle">
        Please ensure the user attached a valid proof document
      </span>
    </div>
    <ul class="accredited-kyc-viewer__proof-document-wrp key-value-list">
      <li>
        <span>Proof document</span>
        <span>
          <user-doc-link-getter
            :file-key="kyc.documents.kycProofInvestor"
          >
            Open file
          </user-doc-link-getter>
        </span>
      </li>
    </ul>

    <general-kyc
      :blob-id="blobId"
      :user="user"
    />
  </div>
</template>

<script>
import GeneralKyc from './UserDetails.GeneralKyc'
import { UserDocLinkGetter } from '@comcom/getters'

export default {
  components: {
    GeneralKyc,
    UserDocLinkGetter,
  },
  props: {
    kyc: {
      type: Object,
      required: true,
    },
    user: {
      type: Object,
      required: true,
    },
    blobId: {
      type: String,
      required: true,
    },
  },
}
</script>

<style lang="scss" scoped>
@import "../../../../../assets/scss/_colors.scss";

.accredited-kyc-viewer__header-title {
  color: $color-text-inverse;
  font-weight: bold;
  margin-bottom: .5rem;
}

.accredited-kyc-viewer__header-subtitle {
  color: $color-text-inverse;
}

.accredited-kyc-viewer__warning-msg {
  padding: 1rem 2.5rem;
  margin-bottom: 2rem;
  display: flex;
  flex-direction: column;
  background-color: $color-banner-bg;
}

.accredited__doc-title {
  margin-top: 1rem;
}

.accredited-kyc-viewer__proof-document-wrp {
  margin-bottom: 2rem;
}
</style>
