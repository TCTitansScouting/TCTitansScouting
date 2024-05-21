<template>
  <div v-show="show">
    <h1 class="page-heading">{{ config.data.heading ?? "Scouting" }}</h1>
    <h3 v-if="teamDesc?.length > 0" class="page-heading">Team: {{ teamDesc }}</h3>

    <video v-if="config.data.logo" ref="videoPlayer" alt="Cannot load logo file" src="animgif.mp4" class="center" autoplay muted @ended="pauseVideo" preload="auto"></video>
    <h2 class="page-heading">{{ title }}</h2>
    <div class="grid">
      <slot></slot>
    </div>
  </div>
</template>

<script setup lang="ts">
  import { Vue, Component, Ref } from 'vue-property-decorator';

  @Component
  export default class VideoComponent extends Vue {
    @Ref('videoPlayer') readonly videoPlayer!: HTMLVideoElement;

    pauseVideo(): void {
      if (this.videoPlayer) {
        this.videoPlayer.pause();
      }
    }
  }
import { useConfigStore, useWidgetsStore } from "@/common/stores";

const props = defineProps<{
  title: string
}>();

const config = useConfigStore();
const widgets = useWidgetsStore();

const teamDesc = $computed(() => widgets.values.find(i => i.name == "Team")?.value.replaceAll(",", ", "));

// Get the full path to the logo image
const absoluteLogoPath = $computed(() => `${import.meta.env.BASE_URL}assets/${config.data.logo}`);

let show = $ref(false);

widgets.lastWidgetRowEnd = 1;

// Expose page data
defineExpose({ title: props.title, setShown: (value: boolean) => show = value });
</script>

<style>
.grid {
  display: grid;
  align-items: center;
  gap: 12px;
}

.center {
  margin-left: auto;
  margin-right: auto;
}

.page-heading {
  text-align: center;
}
</style>
<style scoped>
  #video-container {
    position: relative;
    width: 40%;
    height: 50vh;
    overflow: hidden;
  }
  video {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
</style>


