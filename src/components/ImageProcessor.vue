<script setup lang="ts">
import { ref } from 'vue'

const imageFile = ref<File | null>(null)
const processedImages = ref<string[]>([])

function handleFileUpload(event: Event) {
  const target = event.target as HTMLInputElement
  if (target.files && target.files[0]) {
    imageFile.value = target.files[0]
  }
}

function processImage() {
  if (!imageFile.value)
    return

  const img = new Image()
  img.src = URL.createObjectURL(imageFile.value)

  img.onload = () => {
    const canvas = document.createElement('canvas')
    const ctx = canvas.getContext('2d')

    if (!ctx)
      return

    const { width, height } = img
    const pieceSize = height
    const pieces = Math.ceil(width / pieceSize)

    for (let i = 0; i < pieces; i++) {
      canvas.width = pieceSize
      canvas.height = pieceSize

      // Fill with white before drawing the image
      ctx.fillStyle = '#ffffff'
      ctx.fillRect(0, 0, pieceSize, pieceSize)

      // Draw the image piece on the canvas
      ctx.drawImage(img, -i * pieceSize, 0, width, height)

      const dataUrl = canvas.toDataURL('image/jpeg')
      processedImages.value.push(dataUrl)
    }
  }
}
</script>

<template>
  <div class="flex flex-col items-center justify-center p-10 text-white space-y-4">
    <div>
      <input id="image-input" type="file" accept="image/jpeg" @change="handleFileUpload">
    </div>
    <button :disabled="!imageFile" @click="processImage">
      Process Image
    </button>
    <div v-if="processedImages.length > 0" class="flex flex-wrap gap-2">
      <div v-for="(image, index) in processedImages" :key="index">
        <!-- Use an anchor tag to enable download on click -->
        <a :href="image" :download="`processed_image_${index + 1}.jpg`">
          <img :src="image" :alt="`Processed Image ${index + 1}`">
        </a>
      </div>
    </div>
  </div>
</template>

<style scoped>
input[type='file'] {
  display: block;
  margin-bottom: 10px;
}
button {
  padding: 8px 12px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
button:disabled {
  background-color: #9e9e9e;
  cursor: not-allowed;
}
img {
  max-width: 100px;
  max-height: 100px;
  object-fit: cover;
  cursor: pointer;
}
a {
  text-decoration: none;
}
</style>
