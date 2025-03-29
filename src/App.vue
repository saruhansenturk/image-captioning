<template>
  <div class="flex justify-center items-center h-screen">
    <div class="flex flex-col gap-4 items-center w-96 h-8/12 rounded-lg p-4 shadow-lg relative"
      style="background: rgba(255, 255, 255, 0.2); backdrop-filter: blur(10px); -webkit-backdrop-filter: blur(10px); border: 1px solid rgba(255, 255, 255, 0.3);">

      <!-- Görsel Önizleme Alanı -->
      <div v-if="imageUrl" class="w-80 h-80 bg-gray-200 rounded-lg flex justify-center items-center relative">
        <img :src="imageUrl" alt="Selected Image" class="w-full h-full object-contain" />
      </div>
      <div v-else class="w-80 h-80 bg-gray-200 rounded-lg overflow-hidden flex justify-center items-center">
        <span class="text-gray-500">No image selected</span>
      </div>

      <!-- Drag & Drop ve File Input -->
      <div class="input-group" :class="{ 'border-blue-500 bg-gray-200': isDragging }"
        @dragover.prevent="isDragging = true" @dragleave="isDragging = false" @drop="handleDrop">
        <input type="file" class="form-control" ref="fileInput" @change="handleFileChange" accept="image/*" />
      </div>

      <!-- Caption Butonu -->
      <button class="btn btn-success w-full mt-2" @click="captionToImg">
        <span v-if="isApiProcess" class="spinner-border spinner-border-sm"></span>
        <i v-else class="fas fa-paper-plane"></i> Generate Caption
      </button>

      <!-- Caption Sonucu -->
       <div class="flex w-full h-auto">
         <div class="my-4 bg-gray-700 rounded-lg p-2 text-white">
           <span v-if="isApiProcess" class="flex rounded-full bg-green-300 w-4 h-4 animate-pulse"></span>
           <span v-else>
             {{ caption }}
           </span>
         </div>
       </div>

    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const fileInput = ref<HTMLInputElement | null>(null)
const imageUrl = ref<string | null>(null)
const isDragging = ref(false)
const isApiProcess = ref(false);
const caption = ref('');
const selectedFile = ref<File | null>(null);

const triggerFileInput = () => {
  if (fileInput.value) {
    fileInput.value.click()
  }
}

const handleFileChange = (event: Event) => {
  const target = event.target as HTMLInputElement
  if (target.files && target.files[0]) {
    const file = target.files[0]
    selectedFile.value = file;
    imageUrl.value = URL.createObjectURL(file)
  }
}

const handleDrop = (event: DragEvent) => {
  event.preventDefault()
  isDragging.value = false
  if (event.dataTransfer?.files.length) {
    const file = event.dataTransfer.files[0]
    selectedFile.value = file;
    imageUrl.value = URL.createObjectURL(file)
  }
}

const captionToImg = () => {
  if (!selectedFile.value) {
    alert("Please select an image file first.");
    return;
  }

  const formData = new FormData();
  formData.append('file', selectedFile.value);

  isApiProcess.value = true;

  fetch('https://localhost:7231/image-captioning', {
    method: 'POST',
    body: formData,
  })
    .then(async (response) => {
      if (!response.ok) {
        throw new Error('Failed to fetch caption');
      }
      const data = await response.text();
      caption.value = data;
    })
    .catch((error) => {
      console.error('Error:', error);
      alert('An error occurred while fetching the caption.');
    })
    .finally(() => {
      isApiProcess.value = false;
    });
}
</script>

<style scoped>
header {
  line-height: 1.5;
  max-height: 100vh;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}
</style>
