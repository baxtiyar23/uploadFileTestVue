<script setup lang="ts">
import { uploadSingle } from '@/components/api'
import { getFileType, STATUS, statusIcon } from '@/constants'
import { ref } from 'vue'

interface MultiResList {
  id: string
  file: File
  size: number
  img: string
  name: string
  status: string
  objectName: string
}

const multiResData = ref<MultiResList[]>([])
const maxCount = ref(false)
const errorMessage = ref(false)

interface Props {
  maxSize: number
  maxElementCount: number
  isMultiple: boolean
}
const props = defineProps<Props>()

function multiChange(e: Event) {
  const input = (e.target as HTMLInputElement).files

  if (!input) return

  for (let i = 0; i < input.length; i++) {
    const item = input[i]
    errorMessage.value = item.size > props.maxSize
    const type = getFileType(item)
    multiResData.value.push({
      id: new Date().getTime() + i.toString(),
      file: item,
      size: item.size,
      img: type,
      name: item.name,
      status: '',
      objectName: ''
    })
    if (multiResData.value.length == props.maxElementCount) {
      maxCount.value = true
      return
    }
  }
}

const multipleUpload = async () => {
  if (!multiResData.value) return
  for (const item of multiResData.value) {
    try {
      if (item.status === statusIcon[STATUS.success]) continue
      if (item.size >= props.maxSize) {
        item.status = statusIcon[STATUS.failed]
        console.log('error')
        continue
      }
      const response = await uploadSingle(item.file)
      item.objectName = response.data.objectName
      console.log('uploading')
      if (response.status === 200) {
        console.log('success')
        item.status = statusIcon[STATUS.success]
      }
    } catch (error) {
      item.status = statusIcon[STATUS.error]
    }
  }
}

const remove = async (id: string) => {
  multiResData.value = multiResData.value.filter((e) => e.id !== id)
  if (multiResData.value.length < props.maxElementCount) {
    maxCount.value = false
  }
}

defineExpose({
  multiChange,
  multiResData,
  uploadSingle
})
</script>

<template>
  <div class="grid gap-2 w-[100vw] h-[100%] p-2">
    <p>Multiple Upload</p>
    <div class="flex flex-wrap gap-2">
      <div class="flex flex-wrap" v-for="item in multiResData" key="item.id">
        <div class="relative">
          <img :src="item.img" class="w-[200px] h-[200px] border" alt="" />
          <img class="absolute top-1 left-1 w-[40px]" :src="item.status" alt="" />
          <p class="w-[150px]">{{ item.name }}</p>
          <button
            @click="remove(item.id)"
            class="absolute right-0 top-0 btn bg-red-600 text-[16px] text-white w-[80px] h-[50px] text-[15px] rounded"
          >
            Delete
          </button>
        </div>
      </div>

      <label
        :title="maxCount ? 'max-count' : ''"
        :id="errorMessage ? 'large-file' : ''"
        :data="multiResData.length >= props.maxElementCount ? 'hidden' : undefined"
        class="label-input border h-[200px] relative"
      >
        <input
          v-if="multiResData.length < props.maxElementCount"
          class="hidden"
          type="file"
          :multiple="isMultiple"
          @change="multiChange"
        />
      </label>
    </div>
    <button @click="multipleUpload" class="btn btn-blue w-[200px]">Save</button>
  </div>
</template>

<style scoped>
.btn {
  @apply font-bold py-2 px-4 rounded;
}

.btn-blue {
  @apply bg-blue-500 text-white;
}

.btn-blue:hover {
  @apply bg-blue-700;
}

.label-input {
  width: 200px;
  height: 200px;
  background-image: url('../assets/icons/upload-file-svgrepo-com.svg');
  background-position: center;
  background-size: cover;
}
</style>
./singleUploadType
