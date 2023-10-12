<script setup>
import Report from './components/Report.vue'
import Label from './components/Label.vue'
import { GPU_LIMITS, GPU_FEATURES, GPU_INFOS, GPU_TEXTURE_FORMATS, DESCRIPTIONS } from './constants.js'
import { ref } from 'vue'

const userAgent = navigator.userAgent;
const webgpuSupported = ref(false);
const gpuLimits = ref([]), gpuFeatures = ref([]), gpuInfos = ref([]), gpuFormats = ref([]);

async function main() {
  if (!navigator.gpu) return;
  webgpuSupported.value = true;

  const GPU = await navigator.gpu.requestAdapter({ powerPreference: 'high-performance' });
  if (!GPU) return;
  
  GPU_FEATURES.forEach(feature => {
    gpuFeatures.value.push([feature, GPU.features.has(feature)]);
  });

  const GPUInfo = await GPU.requestAdapterInfo();
  if (!GPUInfo) return;

  GPU_INFOS.forEach(info => {
    gpuInfos.value.push([info, GPUInfo[info]]);
  });
  gpuInfos.value.push(['preferredCanvasFormat', navigator.gpu.getPreferredCanvasFormat()]);
  gpuInfos.value.push(['isFallbackAdapter', GPU.isFallbackAdapter]);

  const GPUDevice = await GPU.requestDevice({ requiredFeatures: [...GPU.features.values()] });
  if (!GPUDevice) return;

  GPU_LIMITS.forEach(limit => {
    gpuLimits.value.push([limit, GPU.limits[limit], GPUDevice.limits[limit]]);
  });

  // Adapted from https://browserleaks.com/webgpu
  for (const format of GPU_TEXTURE_FORMATS) {
    await async function(device, f) {
      try {
        let w = 1, h = 1;
        let a;
        return f.startsWith("bc") || f.startsWith("e") ? (w = 4, h = 4) : f.startsWith("astc") && (a = f.match(/(\d+)x(\d+)/)) && (w = parseInt(a[1], 10), h = parseInt(a[2], 10)),
        device.createTexture({
          size: [w, h],
          format: f,
          usage: GPUTextureUsage.COPY_SRC | GPUTextureUsage.COPY_DST | GPUBufferUsage.RENDER_ATTACHMENT | GPUBufferUsage.STORAGE_BINDING | GPUBufferUsage.TEXTURE_BINDING
        }),
        true
      } catch (e) {
        return false
      }
    }(GPUDevice, format) ? gpuFormats.value.push([format, true]) : gpuFormats.value.push([format, false]);
  }
}

main();
</script>

<template>
  <h1>WebGPU Report</h1>
  <Label v-if="webgpuSupported" color="#71d1ae"><img src="/true.png" width="25" /><h2>WebGPU is supported</h2></Label>
  <Label v-else color="#d17171"><img src="/false.png" width="25" /><h2>WebGPU isn't supported</h2></Label>
  <p id="user-agent"><b>Current User-Agent:</b> {{ userAgent }}</p><hr v-if="gpuInfos.length > 0"/>
  <Report v-if="gpuInfos.length > 0" title="GPU Informations" type="string" :datas="gpuInfos" :columns="['Information', 'Value']" :description="DESCRIPTIONS.infos" /><hr v-if="gpuLimits.length > 0"/>
  <Report v-if="gpuLimits.length > 0" title="GPU Limits" type="number" :datas="gpuLimits" :columns="['Limit name', 'Your GPU limit', 'Default minimal limit']" :description="DESCRIPTIONS.limits" /><hr v-if="gpuFeatures.length > 0"/>
  <Report v-if="gpuFeatures.length > 0" title="GPU Features" type="boolean" :datas="gpuFeatures" :columns="['Feature name', 'Supported']" :description="DESCRIPTIONS.features" /><hr v-if="gpuFormats.length > 0" />
  <Report v-if="gpuFormats.length > 0" title="GPU Texture Formats" type="boolean" :datas="gpuFormats" :columns="['Format name', 'Supported']" :description="DESCRIPTIONS.formats" />
</template>

<style scoped>
h1 {
  font-size: 3.2em;
  line-height: 1.1;
}
#user-agent {
  width: 40%;
  text-align: center;
}
hr {
  border: 1px solid #f2f2f2;
  width: 40%;
}
</style>
