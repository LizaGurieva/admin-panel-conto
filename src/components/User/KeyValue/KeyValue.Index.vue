<template>
  <div class="key-value">
    <h2>Manage key-value storage</h2>

    <div class="key-value__list-wrp">
      <key-value-manager />
    </div>
  </div>
</template>

<script>
import KeyValueManager from './KeyValueManager'

export default {
  components: {
    KeyValueManager,
  },
}
</script>
