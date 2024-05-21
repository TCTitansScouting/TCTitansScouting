<template>
  <FormPage title="Download Data" ref="page">
    <FormGroup :label-type="LabelType.None" :colspan="2" align="center">
      <button @click="clearForm">Save and Clear Form</button>
      <br>
      <button @click="generateQRCode">Generate QR Code</button>
      <br>
      <div v-if="qrCodeUrl">
        <img :src="qrCodeUrl" alt="QR Code">
      </div>
    </FormGroup>
    <FormGroup :label-type="LabelType.None">
      <div style="height: 20px;"></div>
    </FormGroup>
    <FormGroup :label-type="LabelType.None" :colspan="2" align="center">
      <span v-if="widgets.downloadLink === null">No Saved Data</span>
      <a v-else :download="`scouted-${config.name}.csv`" :href="widgets.downloadLink">Download Saved CSV</a>
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
import { defineProps, ref, computed, defineExpose } from "vue";
import FormPage from "./FormPage.vue";
import FormGroup from "./FormGroup.vue";
import { LabelType } from "@/common/shared";
import QrcodeVue from "qrcode.vue";
import { useConfigStore, useWidgetsStore } from "@/common/stores";
import { useRouter } from "vue-router";
import QRCode from 'qrcode';

const config = useConfigStore();
const widgets = useWidgetsStore();

const props = defineProps<{
  data: any;
}>();

const spreadsheetData = ref('');
const qrCodeUrl = ref('');
const excludeHeaders = ref(false);

const router = useRouter();
const page = ref<InstanceType<typeof FormPage>>();
const qrContainer = ref<HTMLDialogElement>();

function generateQRCode() {
  const dataText = "https://www.youtube.com/watch?v=dQw4w9WgXcQ";
  if (dataText) {
    QRCode.toDataURL(dataText, (err, url) => {
      if (err) {
        console.error(err)
      } else {
        qrCodeUrl.value = url;
      }
    })
  } else {
    alert('Please enter some form data.')
  }
}

function clearForm() {
  widgets.save();
  router.go(0);
}

defineExpose({ title: computed(() => page?.value?.title), setShown: computed(() => page?.value?.setShown) });
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