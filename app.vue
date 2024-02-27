<template>
  <main class="fixed inset-0 flex overflow-hidden flex flex-col h-screen">
    <header class="flex overflow-x-auto h-16 border-b">
      <button
        v-for="(day, index) in daysRange"
        :key="day"
        class="h-16 w-16 flex-shrink-0"
        :class="{ 'bg-blue-500 text-white': selectedDate === day }"
        @click="setCurrentDay(day)"
      >
        <p class="text-lg">{{ index + 1 }}</p>
        <p class="text-xs">{{ formatDate(day) }}</p>
      </button>
    </header>
    <section class="flex-1 flex-grow">
      <div v-if="pending">
        <p>Loading...</p>
      </div>
      <div v-else-if="!currentDayTimes">
        <p>Ramadan has not started yet or no data available for this day</p>
      </div>
      <div v-else>
        <p class="text-xl">Selected Date: {{ formatDate(selectedDate) }}</p>
        <div class="flex justify-around w-full mt-4">
          <div>
            <h2 class="font-semibold">Suhoor</h2>
            <p>{{ formatTime(currentDayTimes.first_light) }}</p>
          </div>
          <div>
            <h2 class="font-semibold">Iftaar</h2>
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

// Initialize geolocation and storage for user's location
const { coords, error } = useGeolocation();
const userLocation = useStorage("user-location", { lat: null, lng: null });

// Load or prompt for user location
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

function promptForLocation() {
  // Implement a method to manually input location, save to `userLocation`
  // This can be a modal or a new component where you input and save the location
}

const startDate = ref("2024-03-11");
const endDate = ref("2024-04-09");
const selectedDate = ref(new Date().toISOString().split("T")[0]);

// Reactive API URL based on stored location
const apiUrl = computed(() => {
  return `https://api.sunrisesunset.io/json?lat=${userLocation.value.lat}&lng=${userLocation.value.lng}&date_start=${startDate.value}&date_end=${endDate.value}`;
});

const { data, pending, refresh } = await useFetch(apiUrl);

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
