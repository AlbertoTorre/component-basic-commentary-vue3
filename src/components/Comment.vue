<template>
  <div class="editable-container">

    <div style="position: relative;">
      <div 
        class="editable-div" 
        contenteditable="true"
        @input="handleTextInput"
        @dragover.prevent="isDragging = true"
        @dragleave.prevent="isDragging = false"
        @drop.prevent="handleDrop"
        @paste="handlePaste"
        :class="{ dragging: isDragging }"
        ref="editableDiv"
      />
      <p style="position: absolute; margin-top: -20px; text-align: center; color:#fff; font-size: 13px; width: 100%;">
        Escribe texto o arrastra archivos aquí
      </p>
    </div>
    

    <div class="file-preview" v-if="files.length">
      <div v-for="(file, index) in files" :key="index" class="file-item">
        <div v-if="file.file.type?.startsWith('image/')" class="image-preview">
          <img :src="file.previewUrl" :alt="file.file.name" />
        </div>
        <div class="text-preview">
          <p>{{ file.file.name }}</p>
        </div>
        <button @click="removeFile(index)" class="remove-btn">✖</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const files = ref([]);
const isDragging = ref(false);
const editableDiv = ref('');

const handleDrop = (event) => {
  isDragging.value = false;
  const droppedFiles = Array.from(event.dataTransfer.files);

  droppedFiles.forEach(file => {
    const previewUrl = URL.createObjectURL(file);
    files.value.push({ file, previewUrl });
  });
};

const handlePaste = (event) => {
  const items = event.clipboardData.items;
  let isFilePasted = false;

  for (let item of items) {
    if (item.kind === 'file') {
      const file = item.getAsFile();
      const previewUrl = URL.createObjectURL(file);
      files.value.push({ file, previewUrl });
      isFilePasted = true;
    }
  }

  if (isFilePasted) {
    event.preventDefault();
  }
};

const removeFile = (index) => {
  URL.revokeObjectURL(files.value[index].previewUrl); // Liberar el objeto URL
  files.value.splice(index, 1);
}
</script>

<style scoped>
.editable-container {
  border: 1px solid #ddd;
  padding: 10px;
  border-radius: 5px;
  width: 100%;
  max-width: 600px;
  margin: auto;
  font-family: Arial, sans-serif;
}

.editable-div {
  min-height: 100px;
  min-width: 300px;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius:5px;
  outline: none;
  cursor: text;
  text-align: left;
  transition: background-color 0.2s ease;
}

.editable-div.dragging {
  background-color: #444;
}

.file-preview {
  margin-top: 20px;
}

.file-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 5px;
  border: 1px solid #ccc;
  border-radius: 5px;
  margin-bottom: 5px;
  background-color: #fafafa;
}

.image-preview img {
  max-width: 100px;
  max-height: 100px;
  object-fit: cover;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.remove-btn {
  background: none;
  border: none;
  color: red;
  cursor: pointer;
  font-size: 16px;
}

.text-preview {
  color: #444;
}
</style>
