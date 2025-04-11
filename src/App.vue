<template>
  <v-app>
    <ToolBar></ToolBar>
    <v-main>
      <v-container fluid grid-list-md>
        <AlertBar></AlertBar>
        <router-view></router-view>
      </v-container>
    </v-main>
    <NotificationBar></NotificationBar>
    <LogSheet></LogSheet>
    <ProcessDetails></ProcessDetails>
  </v-app>
</template>

<script setup>
import { onMounted } from "vue";

import ToolBar from "@/components/ToolBar";
import NotificationBar from "@/components/NotificationBar";
import AlertBar from "@/components/AlertBar";

import LogSheet from "@/components/process/Log";
import ProcessDetails from "@/components/process/Details";

import { useAppStore } from "@/stores/app";

const store = useAppStore();

onMounted(() => {
  store.init();
});
</script>
