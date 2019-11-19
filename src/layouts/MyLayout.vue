<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar class="bg-grey-10 text-white shadow-2">
        <q-avatar>
          <img src="https://assets-v2.culturacolectiva.com/img/logotype.svg">
        </q-avatar>
        <q-space />
        <q-input dark borderless v-model="queryText" input-class="text-right" class="q-ml-md" debounce="500" placeholder="Buscar Canal" :loading="isSearching" @input="onSearchChannel()">
          <template v-slot:append>
            <q-icon v-if="queryText === ''" name="search" />
            <q-icon v-else name="clear" class="cursor-pointer" @click="onCleanSearch()" />
          </template>
        </q-input>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <div class="row justify-center">
        <div class="col-10">
          <q-list bordered v-if="!isChannelSelected">
            <q-item clickable v-for="channel in channels" :key="channel.snippet.channelId" @click="onSelectChannel(channel.snippet.channelId)">
              <q-item-section avatar>
                <q-avatar>
                  <img :src="channel.snippet.thumbnails.default.url">
                </q-avatar>
              </q-item-section>
              <q-item-section>{{ channel.snippet.channelTitle }}</q-item-section>
            </q-item>
          </q-list>
          <q-page class="q-pa-md" v-else>
            <div class="row q-mb-md">
              <div class="col-6">
                <q-input v-model="videoQueryText" input-class="text-left" placeholder="Buscar Video" :loading="isVideoSearching" debounce="500" @input="onSearchVideo()">
                  <template v-slot:append>
                    <q-icon v-if="videoQueryText === ''" name="search" />
                    <q-icon v-else name="clear" class="cursor-pointer" @click="onCleanVideoSearch()" />
                  </template>
                </q-input>
              </div>
              <div class="col-5 q-ml-md q-mt-md">
                <q-toggle
                  v-model="isFourCols"
                  label="4 Columnas"
                  @input="onChangeGrid()"
                />
              </div>
            </div>
            <div class="row q-col-gutter-md justify-center">
              <div :class="gridCols" v-for="video in videos" :key="video.snippet.channelId" @click="onShowVideo(video.id.videoId)" style="cursor: pointer;">
                <q-card class="my-card">
                  <q-img
                    :src="video.snippet.thumbnails.high.url"
                    basic
                  >
                    <div class="absolute-bottom text-h6">
                      {{ video.snippet.title }}
                    </div>
                  </q-img>

                  <q-card-section>
                    {{ video.snippet.description }}
                  </q-card-section>
                </q-card>
              </div>
            </div>
          </q-page>
          <div class="row justify-center">
            <q-btn-group rounded>
              <q-btn color="primary" rounded icon="keyboard_arrow_left" />
              <q-btn color="primary" rounded label="1" />
              <q-btn color="primary" rounded icon="keyboard_arrow_right" />
            </q-btn-group>
          </div>
        </div>
      </div>
    </q-page-container>
    <q-ajax-bar
      ref="bar"
      position="bottom"
      color="accent"
      size="9px"
      skip-hijack
    />
    <q-dialog v-model="videoModal" full-height full-width>
      <q-card>
        <q-card-section style="height:100%;">
          <q-video :src="currentVideoURL" style="height:100%;"/>
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-layout>
</template>

<script>
import { axiosInstance } from 'boot/axios'
export default {
  name: 'MyLayout',

  mounted: function () {
  },

  data () {
    return {
      queryText: '',
      videoQueryText: '',
      isSearching: false,
      isVideoSearching: false,
      channels: [],
      videos: [],
      isChannelSelected: false,
      channelId: null,
      gridCols: 'col-lg-4 col-sm-1 col-md-1',
      isFourCols: false,
      videoModal: false,
      currentVideoURL: null,
      pages: [],
      nextPageToken: null
    }
  },

  methods: {
    onShowVideo: function (videoId) {
      this.currentVideoURL = 'https://www.youtube.com/embed/' + videoId
      this.videoModal = true
    },
    onChangeGrid: function () {
      if (this.isFourCols) {
        this.gridCols = 'col-lg-3 col-sm-1 col-md-1'
      } else {
        this.gridCols = 'col-lg-4 col-sm-1 col-md-1'
      }
    },
    onSelectChannel: async function (channelId) {
      this.isChannelSelected = true
      this.channelId = channelId
      await this.searchVideo()
    },
    searchVideo: async function () {
      try {
        let response = await axiosInstance.get('/channels/' + this.channelId + '/videos', {
          params: {
            query: this.videoQueryText
          }
        })
        this.videos = response.data.items
        this.nextPageToken = response.data.etag.nextPageToken
      } catch (e) {
        console.log(e)
      } finally {
        this.isVideoSearching = false
        this.$refs.bar.stop()
      }
    },
    searchChannel: async function () {
      try {
        let response = await axiosInstance.get('/channels', {
          params: {
            query: this.queryText
          }
        })
        this.channels = response.data.items
      } catch (e) {
        console.log(e)
      } finally {
        this.isSearching = false
        this.$refs.bar.stop()
      }
    },
    onSearchChannel: async function () {
      this.isChannelSelected = false
      this.isSearching = true
      this.triggerBar()
      await this.searchChannel()
    },
    onSearchVideo: async function () {
      this.isVideoSearching = true
      this.triggerBar()
      await this.searchVideo()
    },
    onCleanSearch: function () {
      this.isSearching = false
      this.queryText = ''
    },
    onCleanVideoSearch: function () {
      this.isVideoSearching = false
      this.videoQueryText = ''
    },
    triggerBar: function () {
      const bar = this.$refs.bar
      bar.start()
      this.timer = setTimeout(() => {
        if (this.$refs.bar) {
          this.$refs.bar.stop()
        }
      }, Math.random() * 3000 + 1000)
    }
  }
}
</script>
