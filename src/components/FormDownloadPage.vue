<template>
  <FormPage title="Download Data" ref="page">

    <FormGroup :label-type="LabelType.None" :colspan="2" align="center">

      <button @click="qrContainer?.showModal()">Generate QR Code</button>

    </FormGroup>

    <FormGroup :label-type="LabelType.None" :colspan="2" align="center">

      <button @click="clearForm">Save and Clear Form</button>

    </FormGroup>

    <FormGroup :label-type="LabelType.None">

      <div style="height: 20px;"></div>

    </FormGroup>

    <FormGroup :label-type="LabelType.None" :colspan="2" align="center">

      <span v-if="widgets.savedData.size === 0">No Saved Data</span>

      <template v-else>
        <label for="entry-select">Entry</label>
        <select id="entry-select" v-model.number="selectedIdx">
          <option v-for="[i, name] of entries.entries()" :key="i" :value="i">{{ name }}</option>
        </select>
        <button @click="deleteData">Delete</button>
        <button @click="downloadData">Download</button>
        <button @click="clearData">Clear All</button>
      </template>

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

function deleteData() {
  if (widgets.savedData.size === 0) return;
  if (selectedIdx.value >= entries.value.length) return;

  const entryName = entries.value[selectedIdx.value];
  if (!confirm(`Delete the records for entry '${entryName}' permanently?`)) return;

  widgets.savedData.delete(entryName);
}

function downloadData() {
  if (widgets.savedData.size === 0) return;
  if (selectedIdx.value >= entries.value.length) return;

  const entryName = entries.value[selectedIdx.value];
  const entryData = widgets.savedData.get(entryName);
  if (!entryData) return;

  const csvData = `${entryData.header.join(',')}\n${entryData.values.map(record => record.join(',')).join('\n')}`;
  const blob = new Blob([csvData], { type: 'text/csv' });
  const downloadLink = document.createElement('a');
  downloadLink.href = URL.createObjectURL(blob);
  downloadLink.download = `scouted-${config.name}.csv`;
  downloadLink.click();
}

function clearData() {
  if (widgets.savedData.size === 0) return;
  if (!confirm("Clear all saved data permanently?")) return;

  widgets.savedData.clear();
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
