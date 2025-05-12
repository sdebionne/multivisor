<template>
  <v-bottom-sheet v-model="visible">
    <v-toolbar
      dense
      height="32px"
      :color="log.stream === 'err' ? 'orange' : 'blue'"
      :title="title"
    >
      <!-- <h5>{{ title }}</h5>
      <v-spacer></v-spacer> -->
      <!-- <v-tooltip location="bottom"> -->
        <v-switch
          color="indigo"
          style="height: 32px"
          v-model="autoScroll"
        >
        <!-- <v-tooltip>on / off</v-tooltip> -->
        </v-switch>
        <!-- <span>auto-scroll ({{ autoScroll ? "On" : "Off" }})</span>
      </v-tooltip> -->
      <!-- <v-tooltip location="bottom"> -->
        <v-chip slot="activator" color="indigo white--text">
          {{ localSizeStr }}
          <v-btn
            icon
            size="small"
            location="left"
            class="ml-0"
            @click="text = ''"
          >
            <v-icon>delete</v-icon>
          </v-btn>
        </v-chip>
        <!-- <span>Web console log size</span>
      </v-tooltip> -->
      <!-- <v-tooltip location="bottom"> -->
        <v-chip slot="activator" class="bg-indigo text-white">
          {{ sizeStr }}
        </v-chip>
        <!-- <span>Remote log size</span>
      </v-tooltip> -->
      <v-btn icon size="small" @click="maximize = !maximize" class="mr-2">
        <v-icon v-if="maximize">mdi-expand-more</v-icon>
        <v-icon v-else>mdi-expand-less</v-icon>
      </v-btn>
    </v-toolbar>
    <v-progress-linear
      :indeterminate="active"
      height="1"
      class="my-0"
      bg-color="white"
      color="deep-purple"
    >
    </v-progress-linear>
    <v-card>
      <v-card-text
        :style="windowSize"
        id="logContent"
        class="overflow-y-auto bg-black text-white px-2 py-0"
      >
        <pre class="logConsole">
          {{ text }}
        </pre>
      </v-card-text>
    </v-card>
  </v-bottom-sheet>
</template>

<script setup>
import { computed } from "vue";
import { storeToRefs } from "pinia";
import { formatBytes } from "@/multivisor";

import { useAppStore } from "@/stores/app";

const store = useAppStore();

const text = "";
let size = 0;
const maximize = false;
const autoScroll = true;
const eventSource = null;

const { log } = storeToRefs(store);

const visible = computed({
  get() { return log.value.visible },
  set(newValue) { store.setLogVisible(newValue) }
})

const title = computed(() => {
  if (!visible.value) {
    return "";
  }
  return `${log.value.stream === "out" ? "O-log of " : "E-log of "}
  ${log.value.process.name} on ${log.value.process.supervisor}`;
});

const windowSize = computed(() => {
  let h = window.innerHeight;
  return `height: ${maximize ? h - 80 : Math.min(h / 3, 300)}px;`;
});

const sizeStr = computed(() => {
  return formatBytes(size);
});

const localSizeStr = computed(() => {
  return formatBytes(text.length);
});

const active = computed(() => {
  return eventSource && eventSource.readyState < 2;
});

const appendLogMessage = (data) => {
  size = data.size;
  if (data.message) {
    text += data.message;
    /* At 10Mb, cut log to 9Mb */
    if (text.length > 1e7) {
      text = text.substr(-9000000);
    }
  }
  if (autoScroll) {
    let logTag = document.getElementById("logContent");
    setTimeout(() => {
      logTag.scrollTop = logTag.scrollHeight;
    }, 100);
  }
};

const viewLog = () => {
  if (eventSource !== null) {
    text = "";
    size = 0;
    eventSource.close();
  }
  if (!visible) {
    return;
  }
  let newEventSource = new EventSource(
    `/api/process/log/${log.value.stream}/tail/${log.value.process.uid}`,
  );
  newEventSource.onmessage = (event) => {
    let data = JSON.parse(event.data);
    appendLogMessage(data);
  };
  newEventSource.onopen = (event) => {
    console.debug(log.value.stream + " stream opened for " + log.value.process.uid);
  };
  newEventSource.onclose = (event) => {
    eventSource = null;
  };
  newEventSource.onerror = (event) => {
    eventSource.close();
    eventSource = null;
  };
  eventSource = newEventSource;
};

watch(visible, () => { console.log("view log"); viewLog() })
</script>

<style scoped>
.logConsole {
  font-size: small;
}
</style>
