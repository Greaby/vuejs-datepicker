<template>
  <div :class="{ 'input-group': bootstrapStyling }">
    <!-- Calendar Button -->
    <span
      v-if="calendarButton"
      class="vdp-datepicker__calendar-button"
      :class="{ 'input-group-prepend': bootstrapStyling }"
      @click="showCalendar"
      v-bind:style="{ 'cursor:not-allowed;': disabled }"
    >
      <span :class="{ 'input-group-text': bootstrapStyling }">
        <i :class="calendarButtonIcon">
          {{ calendarButtonIconContent }}
          <span v-if="!calendarButtonIcon">&hellip;</span>
        </i>
      </span>
    </span>
    <!-- Input -->
    <input
      :type="inline ? 'hidden' : 'text'"
      :class="computedInputClass"
      :name="name"
      :ref="refName"
      :id="id"
      :value="formattedValue"
      :open-date="openDate"
      :placeholder="placeholder"
      :clear-button="clearButton"
      :disabled="disabled"
      :required="required"
      :readonly="!typeable"
      @click="showCalendar"
      @keyup="parseTypedDate"
      @blur="inputBlurred"
      autocomplete="off"
    />
    <!-- Clear Button -->
    <span
      v-if="clearButton && selectedDate"
      class="vdp-datepicker__clear-button"
      :class="{ 'input-group-append': bootstrapStyling }"
      @click="clearDate()"
    >
      <svg
        width="12"
        height="12"
        viewBox="0 0 12 12"
        xmlns="http://www.w3.org/2000/svg"
      >
        <path
          d="M.566 1.698L0 1.13 1.132 0l.565.566L6 4.868 10.302.566 10.868 0 12 1.132l-.566.565L7.132 6l4.302 4.3.566.568L10.868 12l-.565-.566L6 7.132l-4.3 4.302L1.13 12 0 10.868l.566-.565L4.868 6 .566 1.698z"
        ></path>
      </svg>
    </span>
    <slot name="afterDateInput"></slot>
  </div>
</template>
<script>
import { makeDateUtils } from "../utils/DateUtils";
export default {
  props: {
    selectedDate: Date,
    resetTypedDate: [Date],
    format: [String, Function],
    translation: Object,
    inline: Boolean,
    id: String,
    name: String,
    refName: String,
    openDate: Date,
    placeholder: String,
    inputClass: [String, Object, Array],
    clearButton: Boolean,
    clearButtonIcon: String,
    calendarButton: Boolean,
    calendarButtonIcon: String,
    calendarButtonIconContent: String,
    disabled: Boolean,
    required: Boolean,
    typeable: Boolean,
    bootstrapStyling: Boolean,
    useUtc: Boolean,
  },
  data() {
    const constructedDateUtils = makeDateUtils(this.useUtc);
    return {
      input: null,
      typedDate: false,
      utils: constructedDateUtils,
    };
  },
  computed: {
    formattedValue() {
      if (!this.selectedDate) {
        return null;
      }
      if (this.typedDate) {
        return this.typedDate;
      }
      return typeof this.format === "function"
        ? this.format(this.selectedDate)
        : this.utils.formatDate(
            new Date(this.selectedDate),
            this.format,
            this.translation
          );
    },

    computedInputClass() {
      if (this.bootstrapStyling) {
        if (typeof this.inputClass === "string") {
          return [this.inputClass, "form-control"].join(" ");
        }
        return { "form-control": true, ...this.inputClass };
      }
      return this.inputClass;
    },
  },
  watch: {
    resetTypedDate() {
      this.typedDate = false;
    },
  },
  methods: {
    showCalendar() {
      this.$emit("showCalendar");
    },
    /**
     * Attempt to parse a typed date
     * @param {Event} event
     */
    parseTypedDate(event) {
      // close calendar if escape or enter are pressed
      if (
        [
          27, // escape
          13, // enter
        ].includes(event.keyCode)
      ) {
        this.input.blur();
      }

      if (this.typeable) {
        const typedDate = Date.parse(this.input.value);
        if (!isNaN(typedDate)) {
          this.typedDate = this.input.value;
          this.$emit("typedDate", new Date(this.typedDate));
        }
      }
    },
    /**
     * nullify the typed date to defer to regular formatting
     * called once the input is blurred
     */
    inputBlurred() {
      if (this.typeable && isNaN(Date.parse(this.input.value))) {
        this.clearDate();
        this.input.value = null;
        this.typedDate = null;
      }

      this.$emit("closeCalendar");
    },
    /**
     * emit a clearDate event
     */
    clearDate() {
      this.$emit("clearDate");
    },
  },
  mounted() {
    this.input = this.$el.querySelector("input");
  },
};
// eslint-disable-next-line
</script>
