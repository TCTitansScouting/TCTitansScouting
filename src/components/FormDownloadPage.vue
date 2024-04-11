<template>
  <FormPage title="Download Data" ref="page">

    <FormGroup :label-type="LabelType.None" :colspan="2" align="center">

      <button @click="generateQRCode()">Generate QR Code</button>

    </FormGroup>

    <FormGroup :label-type="LabelType.None" :colspan="2" align="center">

      <button @click="clearForm">Save and Clear Form</button>

    </FormGroup>

    <FormGroup :label-type="LabelType.None">

      <div style="height: 20px;"></div>

    </FormGroup>

    <FormGroup :label-type="LabelType.None" :colspan="2" align="center">

      <span v-if="widgets.savedData.size === 0">No Saved Data</span>

      <a v-else @click="downloadAllData()">Download All Data</a>

    </FormGroup>

    <FormGroup :label-type="LabelType.None" :colspan="2" align="center">

      <RouterLink :to="{ name: 'inspector' }">Data Inspector</RouterLink>

    </FormGroup>

    <FormGroup :label-type="LabelType.None" :colspan="2" align="center">

      <RouterLink :to="{ name: 'home' }">Home</RouterLink>

    </FormGroup>

  </FormPage>

  <dialog ref="qrContainer">

    <div id="qr-dialog-contents">

      <button id="qr-dialog-close" @click="qrContainer?.close">Close</button>

      <div>

        <input type="checkbox" v-model="excludeHeaders" id="exclude-headers" />

        <label for="exclude-headers">Exclude headers in code</label>

      </div>

      <qrcode-vue :value="qrData" level="M" render-as="svg" :size="350" />

    </div>

  </dialog>

</template>

<script setup lang="ts">
import Ajv from "ajv";
import FormPage from "./FormPage.vue";
import FormGroup from "./FormGroup.vue";
import { LabelType } from "@/common/shared";
import { computed } from "vue";
import { ConfigSchema, Widget } from "@/config";
import { defineStore } from "pinia";
import { useConfigStore } from "@/common/stores";
import { useWidgetsStore } from "@/common/stores";
import { useRouter } from "vue-router";
import { useStorage } from "@vueuse/core";
import { isFailed, TBAData } from "./tba";
import { Ref } from "vue";
import validate from "./validate";

const config = useConfigStore();
const widgets = useWidgetsStore();
const router = useRouter();

const entries = computed(() => [...widgets.savedData.keys()]);
let selectedIdx = ref(0);

function generateQRCode() {
  if (widgets.savedData.size === 0) {
    alert("No data to generate QR code.");
    return;
  }

  const allDataString = Array.from(widgets.savedData.values())
    .map(entry => `${entry.header.join(',')}\n${entry.values.map(record => record.join(',')).join('\n')}`)
    .join('\n');

  const qrData = excludeHeaders ? allDataString : allDataString.replace(/,+/g, '\n');

  showQRCodeDialog(qrData);
}

function showQRCodeDialog(qrData: string) {
  // Assuming qrContainer is a ref to the dialog element
  qrContainer?.showModal();

  // Assuming qrDataRef is a ref to the QR code value
  qrDataRef.value = qrData;
}

function downloadAllData() {
  if (widgets.savedData.size === 0) {
    alert("No data to download.");
    return;
  }

  const allDataString = Array.from(widgets.savedData.values())
    .map(entry => `${entry.header.join(',')}\n${entry.values.map(record => record.join(',')).join('\n')}`)
    .join('\n');

  const blob = new Blob([allDataString], { type: 'text/csv' });
  const downloadLink = document.createElement('a');
  downloadLink.href = URL.createObjectURL(blob);
  downloadLink.download = `all-scouted-data.csv`;
  downloadLink.click();
}

function clearForm() {
  widgets.save();
  router.go(0); // Reload the page
}

defineExpose({ title: computed(() => page?.title), setShown: computed(() => page?.setShown) });

</script>

<style lang="postcss">

#qr-dialog-contents {

  display: flex;

  flex-direction: column;

  gap: 4px;

  align-items: flex-start;



  button {

    align-self: flex-end;

  }



  label {

    color: black;

  }

}

</style>
