<template>
  <v-container justify-center>
    <v-data-table
      :headers="headers"
      :items="procs"
      :search="search"
      v-model="selectedProcesses"
      hide-actions
      select-all
      no-results-text="Sorry, no matching processes found"
      no-data-text="Sorry, there are no processes currently being monitored"
      must-sort
      show-select
      item-key="uid"
      class="elevation-4"
    >
      <template v-slot:item="{ item }">
        <ProcessRow
          :process="item"
          :show-supervisor="showSupervisor"
          :show-group="showGroup"
        ></ProcessRow>
      </template>
    </v-data-table>
  </v-container>
</template>

<script setup>
import ProcessRow from "@/components/process/Row";
import { useAppStore } from "@/stores/app";

const store = useAppStore();

const { processes, showGroup, showSupervisor } = defineProps({
  processes: { default: null },
  showGroup: { default: true },
  showSupervisor: { default: true },
});

const headers = computed(() => {
  let header = [
    {
      align: "left",
      sortable: true,
      title: "Name",
      value: "name",
      tooltip: "process name",
    },
  ];
  if (showGroup) {
    header.push({
      align: "left",
      sortable: true,
      title: "Group",
      value: "group",
      tooltip: "process group",
      class: "hidden-xs-only",
    });
  }
  if (showSupervisor) {
    header.push({
      align: "left",
      sortable: true,
      title: "Supervisor",
      value: "supervisor",
      tooltip: "supervisor controlling process",
      class: "hidden-xs-only",
    });
  }
  header.push({
    align: "left",
    sortable: true,
    title: "State",
    value: "statename",
    tooltip: "process state",
  });
  header.push({
    align: "left",
    sortable: false,
    title: "Actions",
    value: "",
    tooltip: "(re)start/stop/view log",
  });
  return header;
});

const search = computed(() => {
  return store.search;
});

const selectedProcesses = computed({
  get() {
    return store.selectedProcesses;
  },
  set(newValue) {
    store.setSelectedProcesses(newValue);
  },
});

const procs = computed(() => {
  return processes || store.processes;
});
</script>
