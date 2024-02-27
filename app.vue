<template>
  <main class="fixed inset-0 flex overflow-hidden flex flex-col h-screen">
    <header
      class="flex overflow-x-auto border-b dark:border-white/10 gap-1 p-1"
    >
      <UButton
        v-for="(day, index) in daysRange"
        :key="day"
        :color="selectedDate === day ? 'primary' : 'gray'"
        class="h-16"
        :class="{ 'active-date': selectedDate === day }"
        :variant="selectedDate === day ? 'solid' : 'ghost'"
        @click="setCurrentDay(day)"
      >
        <div>
          <p class="text-lg">{{ index + 1 }}</p>
          <p class="text-xs">{{ formatDate(day) }}</p>
        </div>
      </UButton>
    </header>
    <section class="flex-1 flex-grow">
      <div v-if="pending">
        <p>Loading...</p>
      </div>
      <div
        v-else-if="!currentDayTimes"
        class="h-full flex items-center justify-center p-6 flex-col text-center gap-6"
      >
        <Icon name="i-solar-moon-broken" class="text-gray-500 h-6 w-6" />
        <p>Ramadan has not started yet or no data available for this day</p>
      </div>
      <div
        v-else
        class="h-full flex flex-col items-center justify-center text-center"
      >
        <p class="text-5xl font-bold">Day {{ dayIndex }}</p>
        <p class="mt-2">{{ formatDate(selectedDate) }}</p>
        <div class="flex justify-around w-full mt-4">
          <div>
            <UIcon name="i-lucide-sunrise" class="text-3xl text-gray-500" />
            <h2 class="font-medium">Suhoor</h2>
            <p>{{ formatTime(currentDayTimes.first_light) }}</p>
          </div>
          <div>
            <UIcon name="i-lucide-sunset" class="text-3xl text-gray-500" />
            <h2 class="font-medium">Iftaar</h2>
            <p>{{ formatTime(currentDayTimes.sunset) }}</p>
          </div>
        </div>
      </div>
    </section>
    <footer
      class="border-t dark:border-white/10 h-16 w-full flex items-center justify-between px-4"
    >
      <UColorModeButton />
      <UButton
        variant="ghost"
        color="gray"
        icon="i-lucide-settings"
        @click="settingsOpen = true"
      />
    </footer>
    <UModal v-model="settingsOpen">
      <div class="p-4 space-y-4">
        <UFormGroup label="Latitude" size="lg">
          <UInput placeholder="you@example.com" />
        </UFormGroup>
        <UFormGroup label="Longitude" size="lg">
          <UInput placeholder="you@example.com" />
        </UFormGroup>
        <UFormGroup label="Ramadan start date" size="lg">
          <UPopover :popper="{ placement: 'bottom-start' }">
            <UButton
              icon="i-heroicons-calendar-days-20-solid"
              :label="format(startDate, 'd MMM, yyy')"
              block
              size="lg"
              color="white"
              class="justify-between"
            />
            <template #panel="{ close }">
              <DatePicker v-model="startDate" @close="close" />
            </template>
          </UPopover>
        </UFormGroup>
        <UFormGroup label="Ramadan end date" size="lg">
          <UPopover :popper="{ placement: 'bottom-start' }">
            <UButton
              icon="i-heroicons-calendar-days-20-solid"
              :label="format(endDate, 'd MMM, yyy')"
              block
              size="lg"
              color="white"
              class="justify-between"
            />
            <template #panel="{ close }">
              <DatePicker v-model="endDate" @close="close" />
            </template>
          </UPopover>
        </UFormGroup>
        <UButton label="Save" block size="lg" />
      </div>
    </UModal>
  </main>
</template>

<script setup>
import { useGeolocation, useStorage } from "@vueuse/core";
import { format } from "date-fns";

const { coords, error } = useGeolocation();
const userLocation = useStorage("user-location", { lat: null, lng: null });
const settingsOpen = ref(false);
const startDate = ref("2024-03-11");
const endDate = ref("2024-04-09");
const selectedDate = ref(getLocalDate());

if (!userLocation.value.lat || !userLocation.value.lng) {
  if (!error.value) {
    watch(
      coords,
      (newCoords) => {
        userLocation.value = {
          lat: newCoords.latitude,
          lng: newCoords.longitude,
        };
      },
      { immediate: true }
    );
  }
}

function getLocalDate() {
  const now = new Date();
  const localDate = new Date(now.getTime() - now.getTimezoneOffset() * 60000)
    .toISOString()
    .split("T")[0];
  return localDate;
}

const apiUrl = computed(() => {
  return `https://api.sunrisesunset.io/json/range?lat=${userLocation.value.lat}&lng=${userLocation.value.lng}&date_start=${startDate.value}&date_end=${endDate.value}`;
});

const { data, pending } = await useFetch(apiUrl);

const dayIndex = computed(() => {
  return daysRange.value.findIndex((day) => day === selectedDate.value) + 1;
});

const daysRange = computed(() => {
  const start = new Date(startDate.value);
  const end = new Date(endDate.value);
  const days = [];
  for (let d = new Date(start); d <= end; d.setDate(d.getDate() + 1)) {
    days.push(new Date(d).toISOString().split("T")[0]);
  }
  return days;
});

const currentDayTimes = computed(() => {
  if (!data.value || !Array.isArray(data.value.results)) return null;
  return (
    data.value.results.find((day) => day.date === selectedDate.value) || null
  );
});

function setCurrentDay(day) {
  selectedDate.value = day;
}

function formatDate(dateString) {
  const date = new Date(dateString);
  return date.toLocaleDateString(undefined, { day: "numeric", month: "short" });
}

function formatTime(timeString) {
  const [hours, minutes, ampm] = timeString.split(":");
  return `${hours}:${minutes} ${ampm}`;
}

onMounted(async () => {
  await nextTick(() => {
    observeActiveDate();
  });
});

function observeActiveDate() {
  const observer = new MutationObserver((mutationsList, observer) => {
    for (let mutation of mutationsList) {
      if (mutation.type === "childList" || mutation.type === "attributes") {
        const el = document.querySelector(".active-date");
        if (el) {
          el.scrollIntoView({
            behavior: "smooth",
            block: "nearest",
            inline: "center",
          });
          observer.disconnect();
          break;
        }
      }
    }
  });

  observer.observe(document.body, {
    childList: true,
    subtree: true,
    attributes: true,
    attributeFilter: ["class"],
  });
}
</script>
