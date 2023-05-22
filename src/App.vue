<template>
  <div id="app">
    <PdfToImageConverter />
  </div>
</template>

<script>
import PdfToImageConverter from './components/PdfToImageConverter.vue'

export default {
  name: 'App',
  components: {
    PdfToImageConverter
  }
}
</script>
