<script setup lang="ts">
import { ref } from 'vue'
import { uploadSingle } from './api'
import { STATUS } from '@/constants/Status'
import { statusIcon } from '@/constants/Icons'

interface FileDetails {
  file: File
  size: number
  img: string
  name: string
  status: {}
  errorMessage: boolean
}
interface Props {
  maxSize: number
  fileType: string
}
const props = defineProps<Props>()
interface FileType {
  type: string
  imgUrl: string
}

const allowedMimeTypes: FileType[] = [
  {
    type: 'pdf',
    imgUrl:
      'https://play-lh.googleusercontent.com/9XKD5S7rwQ6FiPXSyp9SzLXfIue88ntf9sJ9K250IuHTL7pmn2-ZB0sngAX4A2Bw4w'
  },
  {
    type: 'zip',
    imgUrl:
      'https://d1nhio0ox7pgb.cloudfront.net/_img/g_collection_png/standard/512x512/folder_zip.png'
  },
  { type: 'sql', imgUrl: 'https://www.shareicon.net/data/2015/09/07/97430_document_512x512.png' },
  {
    type: 'html',
    imgUrl:
      'https://cdn4.iconfinder.com/data/icons/smashicons-file-types-flat/56/22_-_HTML_File_Flat-512.png'
  }
]

const singleFile = ref<FileDetails>()

const singleChange = (e: Event) => {
  const input = e.target as HTMLInputElement
  if (!input.files) return

  const file = input.files[0]
  const fileType = getFileType(file)

  singleFile.value = {
    file,
    size: file.size,
    img: fileType.imgUrl,
    name: file.name,
    status: STATUS,
    errorMessage: file.size > props.maxSize
  }
}

const getFileType = (file: File): FileType => {
  const fileType = allowedMimeTypes.find((ft) => file.type.includes(ft.type))

  if (fileType) {
    return fileType
  } else if (file.type.includes('image')) {
    return {
      type: 'image',
      imgUrl: URL.createObjectURL(file)
    }
  } else {
    return {
      type: 'other',
      imgUrl: 'https://www.iconpacks.net/icons/2/free-file-icon-1453-thumb.png'
    }
  }
}

const singleUpload = async () => {
  if (!singleFile.value) {
    console.log('no file')
    return
  }

  try {
    if (singleFile.value.size < props.maxSize) {
      console.log('upload')
      await uploadSingle(singleFile.value.file)
      singleFile.value.status = statusIcon[STATUS.success]
      singleFile.value.errorMessage = false
    } else {
      singleFile.value.status = statusIcon[STATUS.error]
      singleFile.value.errorMessage = true
    }
  } catch (error: any) {
    singleFile.value.status = statusIcon[STATUS.failed]
  }
}
defineExpose({
  singleChange,
  getFileType,
  singleUpload,
  uploadSingle
})
</script>

<template>
  <div class="grid gap-2 w-[200px] h-[100%] p-2">
    <p>Single Upload</p>
    <label
      :title="singleFile?.errorMessage ? 'large-file' : undefined"
      :id="singleFile?.img ? 'type-success' : undefined"
      class="buttons border relative"
      :style="{ backgroundImage: `url(${singleFile?.img})` }"
    >
      <img class="absolute top-1 left-1 w-[30px]" :src="singleFile?.status" alt="" />
      <input class="hidden" type="file" @change="singleChange" />
    </label>
    <p class="w-[200px]">{{ singleFile?.name }}</p>
    <button @click="singleUpload" class="btn btn-blue w-[200px]">Update</button>
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

.buttons {
  width: 200px;
  height: 200px;
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
}
</style>
