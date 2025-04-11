<template>
  <v-snackbar
    :timeout="5000"
    location="bottom right"
    :color="color"
    v-model="visible"
  >
    {{ lastNotification.message }}
  </v-snackbar>
</template>

<script setup>
import { ref, watch, computed } from "vue";
import { useAppStore } from "@/stores/app";
import { notificationColorMap } from "@/multivisor";

const store = useAppStore();

let visible = ref(false);
let color = ref("info");

const lastNotification = computed(() => {
  let n = store.notifications.length;
  return n ? store.notifications[n - 1] : { message: "" };
});

watch(lastNotification, (notification) => {
  visible = true;
  color = notificationColorMap[notification.level];
});
</script>
