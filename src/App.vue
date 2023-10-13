<script setup>
import Report from './components/Report.vue'
import Label from './components/Label.vue'
import { GPU_LIMITS, GPU_FEATURES, GPU_INFOS, GPU_TEXTURE_FORMATS, DESCRIPTIONS } from './constants.js'
import { ref } from 'vue'

const userAgent = navigator.userAgent;
const webgpuSupported = ref(false), GPU = ref(null);
const gpuLimits = ref([]), gpuFeatures = ref([]), gpuInfos = ref([]), gpuFormats = ref([]);
const error = ref('');

async function main() {
  if (!navigator.gpu) {
    error.value = 'Cannot find navigator.gpu';
    return;
  }
  webgpuSupported.value = true;

  GPU.value = await navigator.gpu.requestAdapter({ powerPreference: 'high-performance' });
  if (!GPU.value) {
    error.value = 'No adapter returned by navigator.gpu.requestAdapter()'
    return;
  }
  
  GPU_FEATURES.forEach(feature => {
    gpuFeatures.value.push([feature, GPU.value.features.has(feature)]);
  });

  const GPUDevice = await GPU.value.requestDevice({ requiredFeatures: [...GPU.value.features.values()] });
  if (!GPUDevice) {
    error.value = 'No device returned by GPUAdapter.requestDevice()';
    return;
  }

  GPU_LIMITS.forEach(limit => {
    gpuLimits.value.push([limit, GPU.value.limits[limit], GPUDevice.limits[limit]]);
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

  const GPUInfo = await GPU.value.requestAdapterInfo();

  if (GPUInfo) {
    GPU_INFOS.forEach(info => {
      gpuInfos.value.push([info, GPUInfo[info]]);
    });
    gpuInfos.value.push(['preferredCanvasFormat', navigator.gpu.getPreferredCanvasFormat()]);
    gpuInfos.value.push(['isFallbackAdapter', GPU.value.isFallbackAdapter]);
  }
}

main();
</script>

<template>
  <h1>WebGPU Report</h1>
  <Label v-if="webgpuSupported && GPU" color="#71d1ae"><img src="/true.png" width="25" /><h2>WebGPU is supported</h2></Label>
  <div v-else>
    <Label color="#d17171"><img src="/false.png" width="25" /><h2>WebGPU isn't supported</h2></Label>
    <p id="error">{{ error }}</p>
  </div>
  <p id="user-agent"><b>Current User-Agent:</b> {{ userAgent }}</p><hr v-if="gpuInfos.length > 0"/>
  <Report v-if="gpuInfos.length > 0" title="GPU Informations" type="string" :datas="gpuInfos" :columns="['Information', 'Value']" :description="DESCRIPTIONS.infos" /><hr v-if="gpuLimits.length > 0"/>
  <Report v-if="gpuLimits.length > 0" title="GPU Limits" type="number" :datas="gpuLimits" :columns="['Limit name', 'Your GPU limit', 'Default minimal limit']" :description="DESCRIPTIONS.limits">
    <span>Follows the <a href="https://gpuweb.github.io/gpuweb/#supported-limits" target="_blank">exhaustive list of supported limits</a> of the W3C's WebGPU specification.</span>
  </Report><hr v-if="gpuFeatures.length > 0"/>
  <Report v-if="gpuFeatures.length > 0" title="GPU Features" type="boolean" :datas="gpuFeatures" :columns="['Feature name', 'Supported']" :description="DESCRIPTIONS.features">
    <span>Follows the <a href="https://gpuweb.github.io/gpuweb/#feature-index" target="_blank">exhaustive list of features</a> of the W3C's WebGPU specification.</span>
  </Report><hr v-if="gpuFormats.length > 0" />
  <Report v-if="gpuFormats.length > 0" title="GPU Texture Formats" type="boolean" :datas="gpuFormats" :columns="['Format name', 'Supported']" :description="DESCRIPTIONS.formats">
    <span>Follows the <a href="https://gpuweb.github.io/gpuweb/#enumdef-gputextureformat" target="_blank">exhaustive list of texture formats</a> of the W3C's WebGPU specification.</span>
  </Report>
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
#error {
  color: #d17171;
  font-weight: bold;
  text-align: center;
  margin-top: 10px;
}
</style>
