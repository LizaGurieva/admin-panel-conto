<template>
  <div
    class="text-field"
    :class="{
      'text-field--error': errorMessage,
      'text-field--disabled': disabled
    }">
    <span
      class="text-field__label"
      :class="{'text-field__label--hidden': isNoLabel}"
    >
      {{ label }}
    </span>

    <textarea
      class="text-field__input"
      :placeholder="placeholder || ' '"
      :value="value"
      :disabled="disabled"
      :name="name"
      :autofocus="autofocus"
      :maxlength="maxlength"
      :required="required"
      :readonly="readonly"
      :title="title"
      :rows="rows"
      :cols="cols"
      @input="onInput"
    />

    <!-- TODO: add maxlength indicator -->
    <!-- TODO: enter key event handle -->

    <transition name="text-field__err-transition">
      <p class="text-field__err-mes" v-if="errorMessage">
        {{ errorMessage }}
      </p>
    </transition>
  </div>
</template>

<script>
export default {
  components: {
    // components
  },

  props: {
    label: { type: String, default: 'Label' },
    value: { type: [String, Number], default: undefined },
    errorMessage: { type: String, default: undefined },

    // proxies
    autocomplete: { type: String, default: 'off' },
    autofocus: { type: Boolean, default: false },
    disabled: { type: Boolean, default: false },
    name: { type: String, default: undefined },
    placeholder: { type: String, default: undefined },
    required: { type: Boolean, default: true },
    readonly: { type: Boolean, default: false },
    title: { type: [String, Number], default: undefined },
    maxlength: { type: [String, Number], default: undefined },

    // textarea proxies
    rows: { type: [String, Number], default: 4 },
    cols: { type: [String, Number], default: undefined },
  },

  computed: {
    isNoLabel () {
      return this.label === null || this.label === '' || this.label === undefined
    },
  },

  methods: {
    onInput (event) {
      this.$emit('input', event.target.value)
    },
  },
}
</script>

<style lang="scss" scoped>
@import "../../../assets/scss/colors";
@import "./scss/fields-variables";

.text-field__label {
  @include text-font-sizes;
  margin-bottom: .5rem;
  display: block;

  &--hidden {
    margin-bottom: 0;
  }
}

.text-field__input {
  @include text-font-sizes;
  caret-color: $field-color-focused;
  padding: .5rem;
  border: 1px solid $field-color-unfocused;
  width: 100%;
  resize: none;
  display: block;

  &:enabled:not([readonly]):focus {
    border-color: $field-color-focused;
  }

  &:disabled {
    background-color: transparent;
    color: $field-color-unfocused;
    cursor: default;
  }
}

.text-field {
  width: 100%;

  &--disabled {
    & > .text-field__label {
      color: $field-color-unfocused;
    }
  }
}

.text-field__err-mes {
  color: $field-color-error;
  margin-top: $field-error-margin-top;
  font-size: $field-error-font-size;
  line-height: $field-error-line-height;
}

.text-field__err-transition-enter-active {
  animation: text-field__err-transition-keyframes $field-transition-duration
  ease-in-out;
}

.text-field__err-transition-leave-active {
  animation: text-field__err-transition-keyframes $field-transition-duration
  ease-in-out reverse;
}

@keyframes text-field__err-transition-keyframes {
  from {
    max-height: 0;
    margin-top: 0;
    overflow: hidden;
  }
  to {
    max-height: $field-error-font-size * $field-error-line-height;
    margin-top: $field-error-margin-top;
    overflow: hidden;
  }
}
</style>
