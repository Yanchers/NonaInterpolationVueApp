<template>
  <section class="container">
    <h3>Домашняя</h3>

    <div class="d-flex gap-5">
      <form class="col d-flex flex-column shadow border rounded p-3">
        <div class="form-floating mb-3 flex-grow-1">
          <input class="form-control" type="number" placeholder="1" v-model="xValue">
          <label class="form-label">X значение</label>
        </div>
        <div class="form-floating mb-3 flex-grow-1">
          <input class="form-control" type="number" placeholder="2" v-model="yValue">
          <label class="form-label">Y значение</label>
        </div>
        <div v-if="interpolationResult !== 0" class="form-floating mb-3 flex-grow-1">
          <input class="form-control" type="number" placeholder="0" v-model="interpolationResult" readonly>
          <label class="form-label">Ответ</label>
        </div>

        <button class="btn btn-primary" type="submit" @click="processInterpolation">Решить</button>
      </form>

      <form class="col d-flex flex-column shadow border rounded p-3" @submit="sendFile">
        <div class="flex-grow-1 mb-3">
          <label class="form-label">Выберите excel файл для загрузки {{file?.name}}</label>
          <input class="form-control" type="file" placeholder="1" @change="onFileChange">
          <div class="valid-feedback">
            Файл норм!
          </div>
          <div class="invalid-feedback">
            Плохой файл!
          </div>
        </div>
        <div class="flex-grow-1 mb-3">
          <label class="form-label">Выберите таблицу</label>
          <select class="form-control" type="file" @change="onFileSelect" v-model="selectedFile">
            <option v-for="f in excelFiles" :key="f">
              {{f}}
            </option>
          </select>
        </div>

        <button class="btn btn-primary" type="submit" @click="sendFile">
          <span v-if="!isFileUploading">Загрузить</span>
          <span v-if="isFileUploading" class="spinner-border spinner-border-sm"></span>
        </button>
      </form>
    </div>

    <table v-if="selectedTable.length !== 0" class="table table-dark table-bordered mt-3">
      <tr v-for="(arr, i1) in selectedTable" :key="i1">
        <td v-for="(n, i2) in arr" :key="i2" :class="{ 'fw-bold': i1 === 0 || i2 === 0 }">
          {{n}}
        </td>
      </tr>
    </table>
  </section>
</template>

<script lang="ts">
import {ref} from "vue";
import axios, {AxiosError} from 'axios';

export default {
  name: "HomeView",

  setup() {
    const file = ref<File>()
    const isFileUploading = ref<boolean>(false)
    const xValue = ref<number>(0)
    const yValue = ref<number>(0)
    const interpolationResult = ref<number>(0)
    const excelFiles = ref<string[]>([])
    const selectedFile = ref<string>()
    const selectedTable = ref<string[][]>([])


    function onFileChange(event: Event) {
      const target = event.target as HTMLInputElement
      if (target && target.files) {
        file.value = target.files[0]

      }
    }
    function sendFile(event: Event) {
      event.preventDefault()
      if (!file.value) return

      const form = new FormData()
      form.append("excel-file", file.value)

      isFileUploading.value = true
      axios.post(
          'http://26.29.14.210:8080/api/excel/upload',
          form
      )
          .then(r => {
            isFileUploading.value = false
            console.log(r)
            console.log("Uploaded file successfully")
          })
          .catch(er => {
            isFileUploading.value = false
            console.error(er)
          })
    }
    function processInterpolation(event: Event) {
      event.preventDefault()

      axios.post(
          `http://26.29.14.210:8080/api/excel/process/${selectedFile.value}`,
          {
            zeta: yValue.value,
            myu: xValue.value,
          }
      )
          .then(r => {
            console.log(r)
            console.log("Processed successfully")
            interpolationResult.value = parseFloat(r.data)
          })
          .catch((er: AxiosError) => {
            console.error(er.response?.data)
          })
    }
    function getFiles() {
      axios.get(
          `http://26.29.14.210:8080/api/excel/files`
      )
          .then(r => {
            console.log(r)
            excelFiles.value = r.data
            selectedFile.value = r.data[0]
            onFileSelect()
          })
          .catch(er => {
            console.error(er)
          })
    }
    function onFileSelect() {
      selectedTable.value = []
      axios.get(
          `http://26.29.14.210:8080/api/excel/files/${selectedFile.value}`
      )
          .then(r => {
            console.log(r)
            selectedTable.value = r.data as string[][]
          })
          .catch(er => {
            console.error(er)
          })
    }

    getFiles()

    return {
      xValue,
      yValue,

      file,
      onFileChange,
      sendFile,
      isFileUploading,

      processInterpolation,
      interpolationResult,

      excelFiles,

      onFileSelect,
      selectedFile,
      selectedTable,
    }
  },

  methods: {
  },

  data() {
    return {
    }
  }
}
</script>

<style scoped>

</style>