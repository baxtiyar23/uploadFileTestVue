<script lang="ts" setup>
import SingleFileUpload from '@/components/singleFileUpload.vue'
import MultiFileUpload from '@/components/multiFileUpload.vue'
</script>

<template>
  <div class="new-app">
    <SingleFileUpload max-size="10 * 1024 * 1024" file-type="" />
    <MultiFileUpload max-element-count="" max-size="" is-multiple="" />
  </div>
</template>

<style>
.new-app .v-input {
  border-radius: 4px;
  border: 1px solid transparent;
}
</style>
