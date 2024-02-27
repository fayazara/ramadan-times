<template>
  <main class="fixed inset-0 flex overflow-hidden flex flex-col h-screen">
    <header class="flex overflow-x-auto border-b gap-1 p-1">
      <UButton
        v-for="(day, index) in daysRange"
        :key="day"
        :color="selectedDate === day ? 'black' : 'gray'"
        class="h-16"
        :variant="selectedDate === day ? 'solid' : 'soft'"
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
        <Icon name="i-solar-moon-broken" class="text-gray-500 h-6 w-6"/>
        <p>Ramadan has not started yet or no data available for this day</p>
      </div>
      <div
        v-else
        class="h-full flex flex-col items-center justify-center text-center"
      >
        <p class="text-xl font-bold">Day {{ dayIndex }}</p>
        <!-- Add this line -->
        <p>{{ formatDate(selectedDate) }}</p>
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
    <footer class="border-t h-16 w-full flex items-center justify-between px-4">
      <UButton label="Set location" @click="promptForLocation" />
      <UButton label="Set Ramadan Start and End dates" />
    </footer>
  </main>
</template>

<script setup>
import { useGeolocation, useStorage } from "@vueuse/core";

const { coords, error } = useGeolocation();
const userLocation = useStorage("user-location", { lat: null, lng: null });

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

const dayIndex = computed(() => {
  return daysRange.value.findIndex((day) => day === selectedDate.value) + 1;
});

function promptForLocation() {
  // need to implement this
}

const startDate = ref("2024-03-11");
const endDate = ref("2024-04-09");
const selectedDate = ref(new Date().toISOString().split("T")[0]);

const apiUrl = computed(() => {
  return `https://api.sunrisesunset.io/json?lat=${userLocation.value.lat}&lng=${userLocation.value.lng}&date_start=${startDate.value}&date_end=${endDate.value}`;
});

const { data, pending } = await useFetch(apiUrl);

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
  if (!data.value) return null;
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
  const [hours, minutes] = timeString.split(":");
  return `${hours}:${minutes}`;
}
</script>
