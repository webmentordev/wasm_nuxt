<template>
  <div class="flex h-screen items-center justify-center">
    <div class="flex flex-col">
      <div class="text-center flex flex-col">
        <h1 class="font-semibold text-4xl">WASM + Nuxt</h1>
        <p class="text-lg">Video to Gif Converter</p>
      </div>
      <video class="mb-3" v-show="video" :src="fileURL" controls width="550"></video>
        <form @submit.prevent="uploadHandler">
            <input type="file" @change="updateFile">
            <button @click="convertToGif" class="py-2 px-4 bg-indigo-600 text-white rounded-sm">Convert</button>
        </form>
      <img v-show="gif" :src="gif" width="550">
    </div>
  </div>
</template>

<script setup>
  import { createFFmpeg, fetchFile } from '@ffmpeg/ffmpeg';

  const ffmpeg = createFFmpeg({ log: true });

  const ready = ref(false);
  const video = ref();
  const fileURL = ref();
  const gif = ref();

  const load = async () => {
    await ffmpeg.load()
    ready.value = true;
  }

  onMounted(() => {
    load()
  })

  const updateFile = (event) => {
    video.value = event.target.files?.item(0)
    fileURL.value = URL.createObjectURL(video.value)
  }

  const convertToGif = async () => {
    ffmpeg.FS('writeFile', 'test.mp4', await fetchFile(video.value));
    await ffmpeg.run('-i', 'text.mp4', '-t', '2.5', '-ss', '2.0', 'out.gif');
    const data = ffmpeg.FS('readFile', 'out.gif')

    const url = URL.createObjectURL(new Blob([data.buffer], { type: 'image/gif' }));
    gif.value = url;
  }

</script>
