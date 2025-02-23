<script setup lang="ts">
import AdminMenu from "@/components/Game/AdminMenu/Base.vue";
import romApi from "@/services/api/rom";
import storeAuth from "@/stores/auth";
import storeDownload from "@/stores/download";
import type { DetailedRom } from "@/stores/roms";
import type { Events } from "@/types/emitter";
import { getDownloadLink, isEmulationSupported } from "@/utils";
import type { Emitter } from "mitt";
import { inject, ref } from "vue";

const props = defineProps<{ rom: DetailedRom }>();
const downloadStore = storeDownload();
const emitter = inject<Emitter<Events>>("emitter");
const auth = storeAuth();
const emulation = ref(false);
const playInfoIcon = ref("mdi-play");
const emulationSupported = isEmulationSupported(props.rom.platform_slug);

function toggleEmulation() {
  emulation.value = !emulation.value;
  playInfoIcon.value = emulation.value ? "mdi-information" : "mdi-play";
  emitter?.emit("showEmulation", null);
}

async function copyDownloadLink(rom: DetailedRom) {
  const downloadLink =
    location.protocol +
    "//" +
    location.host +
    encodeURI(
      getDownloadLink({
        rom,
        files: downloadStore.filesToDownloadMultiFileRom,
      })
    );
  if (navigator.clipboard && window.isSecureContext) {
    await navigator.clipboard.writeText(downloadLink);
    emitter?.emit("snackbarShow", {
      msg: "Download link copied to clipboard!",
      icon: "mdi-check-bold",
      color: "green",
      timeout: 2000,
    });
  } else {
    emitter?.emit("showCopyDownloadLinkDialog", downloadLink);
  }
}
</script>

<template>
  <v-row no-gutters>
    <v-col>
      <v-btn
        @click="
          romApi.downloadRom({
            rom,
            files: downloadStore.filesToDownloadMultiFileRom,
          })
        "
        :disabled="downloadStore.value.includes(rom.id)"
        rounded="0"
        color="primary"
        block
      >
        <v-icon icon="mdi-download" size="large" />
      </v-btn>
    </v-col>
    <v-col>
      <v-btn @click="copyDownloadLink(rom)" rounded="0" color="primary" block
        ><v-icon icon="mdi-content-copy" size="large"
      /></v-btn>
    </v-col>
    <v-col>
      <v-tooltip
        class="tooltip"
        text="Emulation not currently supported"
        location="bottom"
        :disabled="emulationSupported"
      >
        <template v-slot:activator="{ props }">
          <div v-bind="props">
            <v-btn
              rounded="0"
              block
              @click="toggleEmulation"
              :disabled="!emulationSupported"
            >
              <v-icon :icon="playInfoIcon" size="large" />
            </v-btn>
          </div>
        </template>
      </v-tooltip>
    </v-col>
    <v-col>
      <v-menu location="bottom">
        <template v-slot:activator="{ props }">
          <v-btn
            :disabled="!auth.scopes.includes('roms.write')"
            v-bind="props"
            rounded="0"
            block
          >
            <v-icon icon="mdi-dots-vertical" size="large" />
          </v-btn>
        </template>
        <admin-menu :rom="rom" />
      </v-menu>
    </v-col>
  </v-row>
</template>

<style scoped>
.tooltip :deep(.v-overlay__content) {
  background: rgba(201, 201, 201, 0.98) !important;
  color: rgb(41, 41, 41) !important;
}
</style>
