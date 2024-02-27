<script setup lang="ts">
import { DatePicker as VCalendarDatePicker } from "v-calendar";
import { format } from 'date-fns';

import type {
  DatePickerDate,
  DatePickerRangeObject,
} from "v-calendar/dist/types/src/use/datePicker";
import "v-calendar/dist/style.css";

const props = defineProps({
  modelValue: {
    type: [Date, Object] as PropType<
      DatePickerDate | DatePickerRangeObject | null
    >,
    default: null,
  },
});

const emit = defineEmits(["update:model-value", "close"]);

const date = computed({
  get: () => props.modelValue,
  set: (value) => {
    // Check if value is a Date object and format it
    if (value instanceof Date) {
      const formattedDate = format(value, "yyyy-MM-dd");
      emit("update:model-value", formattedDate);
    } else if (value && typeof value === "object" && value.start && value.end) {
      // Assuming value is a range object with start and end dates
      const formattedStart = format(value.start, "yyyy-MM-dd");
      const formattedEnd = format(value.end, "yyyy-MM-dd");
      emit("update:model-value", { start: formattedStart, end: formattedEnd });
    } else {
      // Directly emit the value if it's not a Date or range object
      emit("update:model-value", value);
    }
    emit("close");
  },
});

const attrs = {
  transparent: true,
  borderless: true,
  color: "primary",
  "is-dark": { selector: "html", darkClass: "dark" },
  "first-day-of-week": 2,
};
</script>

<template>
  <VCalendarDatePicker
    v-if="date && typeof date === 'object'"
    v-model.range="date"
    :columns="2"
    v-bind="{ ...attrs, ...$attrs }"
  />
  <VCalendarDatePicker v-else v-model="date" v-bind="{ ...attrs, ...$attrs }" />
</template>

<style>
:root {
  --vc-gray-50: rgb(var(--color-gray-50));
  --vc-gray-100: rgb(var(--color-gray-100));
  --vc-gray-200: rgb(var(--color-gray-200));
  --vc-gray-300: rgb(var(--color-gray-300));
  --vc-gray-400: rgb(var(--color-gray-400));
  --vc-gray-500: rgb(var(--color-gray-500));
  --vc-gray-600: rgb(var(--color-gray-600));
  --vc-gray-700: rgb(var(--color-gray-700));
  --vc-gray-800: rgb(var(--color-gray-800));
  --vc-gray-900: rgb(var(--color-gray-900));
}

.vc-primary {
  --vc-accent-50: rgb(var(--color-primary-50));
  --vc-accent-100: rgb(var(--color-primary-100));
  --vc-accent-200: rgb(var(--color-primary-200));
  --vc-accent-300: rgb(var(--color-primary-300));
  --vc-accent-400: rgb(var(--color-primary-400));
  --vc-accent-500: rgb(var(--color-primary-500));
  --vc-accent-600: rgb(var(--color-primary-600));
  --vc-accent-700: rgb(var(--color-primary-700));
  --vc-accent-800: rgb(var(--color-primary-800));
  --vc-accent-900: rgb(var(--color-primary-900));
}
</style>
