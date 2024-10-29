<template>
  <div>
    <div v-if="showLabel" :class="labelClasses">
      {{ df.label }}
    </div>
    <input
      v-show="true"
      ref="input"
      :class="[inputClasses, dateContainerClasses, ,'date-picker' ]"
      :type="inputType"
      :value="inputValue"
      :placeholder="inputPlaceholder"
      :readonly="isReadOnly"
      @blur="onBlur"
      @focus="onFocus"
      @input="(e) => $emit('input', e)"
    />
   
  </div>
</template>
<script lang="ts">
import { DateTime } from 'luxon';
import { fyo } from 'src/initFyo';
import { defineComponent, nextTick } from 'vue';
import Base from './Base.vue';
// @ts-ignore
import NepaliDatePicker from '@anuz-pandey/nepali-date-picker'
import NepaliDate from 'nepali-date-converter'


export default defineComponent({
  extends: Base,
  emits: ['input', 'focus'],
  data() {
    return {
      showInput: false,
      NepaliDatePickerObject: null,
    };
  },
  computed: {
    inputValue(): string {
      let value = this.value;
      if (typeof value === 'string') {
        value = new Date(value);
      }
      
      try{
        const [year, month, date] = DateTime.fromJSDate(value).toISODate().split("-")
        let date1 = new NepaliDate(new Date(year, date, month))
  
        value = new Date(date1.getYear(), date1.getMonth(), date1.getDay());
      }catch(e){}

      if (value instanceof Date && !Number.isNaN(value.valueOf())) {
        return DateTime.fromJSDate(value).toISODate();
      }

      return '';
    },
    inputType() {
      return 'date';
    },
    formattedValue() {
      const value = this.parse(this.value);
      return fyo.format(value, this.df, this.doc);
    },
    borderClasses(): string {
      if (!this.border) {
        return '';
      }

      const border = 'border border-gray-200 dark:border-gray-800';
      let background = 'bg-gray-25 dark:bg-gray-875';
      if (this.isReadOnly) {
        background = 'bg-gray-50 dark:bg-gray-850';
      }

      if (this.showInput) {
        return background;
      }

      return border + ' ' + background;
    },
  },
  mounted() {
      // Initialize the Nepali date picker on the input element when the component mounts
      const nepaliDateInput = this.$refs.input;
      if (nepaliDateInput) {
        // @ts-ignore
        this.NepaliDatePickerObject = new NepaliDatePicker('.date-picker');
      }
    },
  methods: {
    onFocus(e: FocusEvent) {
      const target = e.target;
      if (!(target instanceof HTMLInputElement)) {
        return;
      }

      target.select();
      this.showInput = true;
      this.$emit('focus', e);
    },
    onBlur(e: FocusEvent) {
      const target = e.target;
      if (!(target instanceof HTMLInputElement)) {
        return;
      }
      this.showInput = false;

      let value: Date | null = DateTime.fromISO(target.value).toJSDate();
      if (Number.isNaN(value.valueOf())) {
        value = null;
      }

      this.triggerChange(value);
    },
    activateInput() {
      if (this.isReadOnly) {
        return;
      }

      this.showInput = true;
      nextTick(() => {
        this.focus();

        // @ts-ignore
        this.$refs.input = this.NepaliDatePickerObject;
      });
    },
  },
});
</script>
