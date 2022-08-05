<template>
  <v-container>
    <v-row>
      <v-col>
        <v-card>
          <card-title :icon="cardIcon" :title="cardTitle"></card-title>
          <v-card-subtitle> {{ cardSubtitle }}</v-card-subtitle>
          <v-card-text>
            <v-container v-if="type === `image`">
              <v-row>
                <v-col
                  cols="2"
                  v-for="(file, index) in resultFiles"
                  :key="index"
                >
                  <v-img
                    :src="file.path"
                    aspect-ratio="1"
                    color="primary"
                    max-height="100"
                    max-width="100"
                  >
                    <template v-slot:placeholder>
                      <v-row
                        class="fill-height ma-0"
                        align="center"
                        justify="center"
                      >
                        <v-progress-circular
                          indeterminate
                          color="primary"
                        ></v-progress-circular>
                      </v-row>
                    </template>
                  </v-img>
                </v-col>
              </v-row>
            </v-container>
            <v-container v-else>
              <v-row>
                <v-col
                  cols="12"
                  v-for="(file, index) in resultFiles"
                  :key="index"
                >
                  <a :href="file.path" download target="_blank">{{
                    file.name
                  }}</a>
                </v-col>
              </v-row>
            </v-container>

            <v-file-input
              :loading="this.uploadSnackbar.status"
              :disabled="this.uploadSnackbar.status"
              v-model="imagesToUpload"
              accept="image/png, image/jpeg, image/bmp"
              :label="inputLabel"
              outlined
              counter
              multiple
              show-size
              truncate-length="15"
              @change="uploadFile"
              small-chips
              dense
              :validate-on-blur="true"
            >
              <template v-slot:selection="{ file, text }">
                <v-chip
                  close
                  @click:close="removeFile(file)"
                  small
                  label
                  color="primary"
                >
                  {{ text }}
                </v-chip>
              </template>
            </v-file-input>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script lang="ts">
import { defineComponent } from '@vue/composition-api'
import BaseMixin from './mixins/base';
import { Vue, Component, mixins, Watch, Prop } from 'nuxt-property-decorator'
import { FileStateFile } from "@/store/files/types";
import { IFile } from '~/store/server/libraryItems/types'

interface uploadSnackbar {
  status: boolean
  filename: string
  percent: number
  speed: number
  total: number
  number: number
  max: number
  cancelTokenSource: any
  lastProgress: {
    time: number
    loaded: number
  }
}

interface draggingFile {
  status: boolean
  item: FileStateFile
}

@Component
export default class FilesUpload extends mixins(BaseMixin) {
  @Prop({ type: String, default: '' }) cardTitle!: string
  @Prop({ type: String, default: '' }) cardSubtitle!: string
  @Prop({ type: String, default: '' }) inputLabel!: string
  @Prop({ type: String, default: '' }) cardIcon!: string
  @Prop({ type: Array, default: [] }) files!: IFile[]
  @Prop({ type: String, default: '' }) type!: string


  resultFiles = this.files
  $refs!: {
    fileUpload: HTMLInputElement,
  }

  private uploadSnackbar: uploadSnackbar = {
    status: false,
    filename: "",
    percent: 0,
    speed: 0,
    total: 0,
    number: 0,
    max: 0,
    cancelTokenSource: {},
    lastProgress: {
      time: 0,
      loaded: 0
    }
  }
  private currentPath = ''

  uploading: boolean = false
  imagesToUpload: File[] = []

  async uploadFile () {
    this.uploading = true

    if (this.imagesToUpload?.length) {
      this.$store.dispatch('socket/addStorageLoading', { name: 'fileUpload' })
      let successFiles = []
      this.uploadSnackbar.number = 0
      this.uploadSnackbar.max = this.imagesToUpload.length
      for (const file of this.imagesToUpload) {
        this.uploadSnackbar.number++
        const result = await this.doUploadFile(file)
        successFiles.push(result)
      }

      this.$store.dispatch('socket/removeStorageLoading', { name: 'fileUpload' })
      for (const file of successFiles) {
        this.$notify({
          channel: 'toast',
          type: 'success',
          text: this.$t('Files.SuccessfullyUploaded', { filename: file }).toString()
        })

        // todo После успешной загрузки отобразить фото на странице
        this.resultFiles.push(
          {
            path: this.apiStorageUrl + '/server/files/library_items/' + file,
            name: this.uploadSnackbar.filename
          })
        this.uploading = false
      }
      console.log('imagesToUpload ', this.imagesToUpload);
      this.imagesToUpload = []

      // this.$refs.fileUpload.innerHTML = '' // .value = '' //todo вызывает ошибку
    }
  }

  doUploadFile (file: File) {
    // создание пустой формы
    let formData = new FormData()
    let filename = file.name.replace(" ", "_")
    this.uploadSnackbar.filename = filename
    this.uploadSnackbar.status = true
    this.uploadSnackbar.percent = 0
    this.uploadSnackbar.speed = 0
    this.uploadSnackbar.lastProgress.loaded = 0
    this.uploadSnackbar.lastProgress.time = 0


    // Добавление нового текстового поля в форму:
    formData.append('file', file, (this.currentPath + "/" + filename).substring(7))
    formData.append('root', 'library_items')

    return new Promise(resolve => {
      this.uploadSnackbar.cancelTokenSource = this.$axios.CancelToken.source();
      this.$axios.post(this.apiStorageUrl + '/server/files/upload',
        formData, {
        cancelToken: this.uploadSnackbar.cancelTokenSource.token,
        headers: { 'Content-Type': 'multipart/form-data' },
        onUploadProgress: (progressEvent: any) => {
          this.uploadSnackbar.percent = (progressEvent.loaded * 100) / progressEvent.total
          if (this.uploadSnackbar.lastProgress.time) {
            const time = progressEvent.timeStamp - this.uploadSnackbar.lastProgress.time
            const data = progressEvent.loaded - this.uploadSnackbar.lastProgress.loaded
            if (time) this.uploadSnackbar.speed = data / (time / 1000)
          }
          this.uploadSnackbar.lastProgress.time = progressEvent.timeStamp
          this.uploadSnackbar.lastProgress.loaded = progressEvent.loaded
          this.uploadSnackbar.total = progressEvent.total
        }
      }
      ).then((result: any) => {
        const filename = result.data.item.path.substr(result.data.item.path.indexOf("/") + 1)
        this.uploadSnackbar.status = false
        resolve(filename)
      }).catch(() => {
        this.uploadSnackbar.status = false
        this.$store.dispatch('socket/removeStorageLoading', { name: 'fileUpload' })
        this.$notify({
          channel: 'toast',
          type: 'error',
          text: this.$t("File.CannotUpload").toString()
        })
      })
    })
  }

  fileRules = [
    (value: any) => {
      let sum = 0
      Array(value).forEach((f: any) => {
        sum += f.size
      })
      return sum < 20000000 || this.$t('Parts.SizeLimit')
    },
    (value: any) => value.length < 9 || this.$t('Parts.CountLimit')
  ]

  imageRules = [
    (value: any) => {
      let sum = 0
      Array(value).forEach((f: any) => {
        sum += f.size
      })
      return sum < 20000000 || this.$t('Parts.SizeLimit')
    },
    (value: any) => value.length < 9 || this.$t('Parts.CountLimit')
  ]

  rules = {
    required: (value: string) => !!value || this.$t('Common.Required'),
    number: (value: string) => String(value).length === String(value).replace(/[^.\d]+/g, "").length || this.$t('Common.RulesNumber'),
  }
}


</script>
