<template>
    <div>
      <input type="file" @change="handleFileChange" accept=".pdf" />
      <div v-if="imageDataList.length">
        <div v-for="(imageData, index) in imageDataList" :key="index">
          <img :src="imageData" :alt="'Page ' + (index + 1)" />
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import { getDocument, GlobalWorkerOptions } from 'pdfjs-dist/build/pdf'
  
  export default {
    data() {
      return {
        imageDataList: []
      }
    },
    methods: {
      async handleFileChange(event) {
        const file = event.target.files[0]
  
        // Create a file reader
        const reader = new FileReader()
  
        // Read the file as ArrayBuffer
        reader.readAsArrayBuffer(file)
  
        // Wait for the file to be loaded
        await new Promise(resolve => {
          reader.onload = resolve
        })
  
        // Get the array buffer of the file
        const arrayBuffer = reader.result
  
        // Load the PDF file using pdf.js
        const loadingTask = getDocument(arrayBuffer)
        const pdf = await loadingTask.promise
  
        const numPages = pdf.numPages
        const imageDataList = []
  
        for (let pageNumber = 1; pageNumber <= numPages; pageNumber++) {
          // Convert each page of the PDF to an image
          const page = await pdf.getPage(pageNumber)
          const viewport = page.getViewport({ scale: 1.0 })
          const canvas = document.createElement('canvas')
          const context = canvas.getContext('2d')
          canvas.height = viewport.height
          canvas.width = viewport.width
  
          const renderTask = page.render({
            canvasContext: context,
            viewport: viewport
          })
  
          await renderTask.promise
  
          // Convert the canvas to base64 image data
          const imageDataURL = canvas.toDataURL()
          imageDataList.push(imageDataURL)
        }
  
        this.imageDataList = imageDataList
      }
    },
    mounted() {
      GlobalWorkerOptions.workerSrc = require('pdfjs-dist/build/pdf.worker.entry')
    }
  }
  </script>
  