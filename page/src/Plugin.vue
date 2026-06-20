<template>
  <div>
    <h2 class="mb-4">{{ $t('plugin_bluez.title') }}</h2>

    <v-skeleton-loader v-if="loading" :loading="true" type="card" />

    <div v-else style="margin-bottom: 80px">
      <!-- Status Card -->
      <v-card class="mb-4 pa-0">
        <v-card-title class="d-flex align-center">
          <v-icon class="mr-2">mdi-bluetooth</v-icon>
          <span>{{ $t('plugin_bluez.status') }}</span>
        </v-card-title>
        <v-card-text>
          <v-row align="center">
            <v-col cols="12" md="3">
              <span class="text-subtitle-1 font-weight-medium">D-Bus</span>
              <v-chip
                size="small" class="ml-2"
                :color="dbusRunning ? 'green' : 'grey'" variant="flat">
                {{ dbusRunning ? $t('plugin_bluez.running') : $t('plugin_bluez.not_running') }}
              </v-chip>
            </v-col>
            <v-col cols="12" md="3">
              <span class="text-subtitle-1 font-weight-medium">Bluetooth</span>
              <v-chip
                size="small" class="ml-2"
                :color="bluetoothRunning ? 'green' : 'grey'" variant="flat">
                {{ bluetoothRunning ? $t('plugin_bluez.running') : $t('plugin_bluez.not_running') }}
              </v-chip>
            </v-col>
          </v-row>
        </v-card-text>
      </v-card>

      <!-- Start/Stop Card -->
      <v-card class="mb-4 pa-0">
        <v-card-title class="d-flex align-center">
          <v-icon class="mr-2">mdi-cog</v-icon>
          <span>{{ $t('plugin_bluez.services') }}</span>
        </v-card-title>
        <v-card-text class="d-flex align-center ga-2">
          <v-btn color="primary" rounded :loading="starting" @click="startServices">
            <v-icon start>mdi-play</v-icon>
            {{ $t('plugin_bluez.start') }}
          </v-btn>
          <v-btn color="error" rounded variant="outlined" :loading="stopping" @click="stopServices">
            <v-icon start>mdi-stop</v-icon>
            {{ $t('plugin_bluez.stop') }}
          </v-btn>
        </v-card-text>
      </v-card>
    </div>

    <!-- Overlay -->
    <v-overlay :model-value="overlay" class="align-center justify-center">
      <v-progress-circular color="onPrimary" size="64" indeterminate />
    </v-overlay>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';

const PLUGIN_NAME = 'bluez';

const loading = ref(true);
const overlay = ref(false);
const starting = ref(false);
const stopping = ref(false);
const statusInterval = ref(null);

const dbusRunning = ref(false);
const bluetoothRunning = ref(false);

const getAuthHeaders = () => ({
  Authorization: 'Bearer ' + localStorage.getItem('authToken'),
});

const runCommand = async (action, timeout = 30, parseJson = false) => {
  return fetch('/api/v1/mos/plugins/query', {
    method: 'POST',
    headers: {
      ...getAuthHeaders(),
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      command: PLUGIN_NAME,
      args: [action],
      timeout,
      parse_json: parseJson,
    }),
  });
};

const checkStatus = async () => {
  try {
    const res = await runCommand('status', 5, true);
    if (res.ok) {
      const data = await res.json();
      if (data.success && data.output) {
        dbusRunning.value = data.output.dbus === true;
        bluetoothRunning.value = data.output.bluetooth === true;
      }
    }
  } catch (e) {
    dbusRunning.value = false;
    bluetoothRunning.value = false;
  }
};

const startServices = async () => {
  starting.value = true;
  try {
    await runCommand('start', 30, false);
    await checkStatus();
  } catch (e) {
    console.error('Failed to start services:', e);
  } finally {
    starting.value = false;
  }
};

const stopServices = async () => {
  stopping.value = true;
  try {
    await runCommand('stop', 30, false);
    await checkStatus();
  } catch (e) {
    console.error('Failed to stop services:', e);
  } finally {
    stopping.value = false;
  }
};

onMounted(async () => {
  try {
    await checkStatus();
    statusInterval.value = setInterval(async () => {
      await checkStatus();
    }, 5000);
  } catch (e) {
    console.error('Failed to initialize:', e);
  } finally {
    loading.value = false;
  }
});

onUnmounted(() => {
  if (statusInterval.value) {
    clearInterval(statusInterval.value);
  }
});
</script>
