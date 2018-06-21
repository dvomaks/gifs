<template>
  <v-app dark>
    <v-toolbar fixed app>
      <v-toolbar-title v-text="title"></v-toolbar-title>
      <v-spacer></v-spacer>
      <v-text-field prepend-icon="search" hide-details single-line v-model="search"></v-text-field>
      <v-spacer></v-spacer>
    </v-toolbar>
    <v-content>
      <v-container>
        <v-layout>
          <v-flex xs12>
            <v-card>
              <v-container grid-list-sm fluid>
                <v-layout wrap>
                  <v-flex
                      v-for="gif in gif_list"
                      :key="gif.id"
                      xs3
                  >
                    <v-card flat tile>
                      <v-card-media
                          :src="gif.images.preview_gif.url"
                          height="200px"
                      >
                      </v-card-media>
                    </v-card>
                  </v-flex>
                </v-layout>
              </v-container>
            </v-card>
          </v-flex>
        </v-layout>
        <infinite-loading @infinite="infiniteHandler" ref="infiniteLoading"></infinite-loading>
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
  import axios from 'axios'
  import InfiniteLoading from 'vue-infinite-loading'
  import _ from 'lodash'

  export default {
    components: {InfiniteLoading},
    data () {
      return {
        search: null,
        title: 'Test for Fleetko',
        loader: true,
        apiKey: 'aYK4iGJrlVQBsXEBOmuXhmx4SKAlmHqL',
        host: 'api.giphy.com',
        perPage: 20,
        gif_list: []
      }
    },
    created(){
      this.debounceOnSearch = _.debounce(this.onSearch, 500)
    },
    watch: {
      search(){
        this.debounceOnSearch()
      }
    },
    methods: {
      onSearch(){
        console.log('test')
        this.gif_list = []
        this.$nextTick(()=>{
          this.$refs.infiniteLoading.$emit('$InfiniteLoading:reset')
        })
      },
      infiniteHandler ($state) {
        const url = this.reqUrl()
        const offset = this.gif_list.length
        axios.get(url, {
          params: {
            api_key: this.apiKey,
            limit: this.perPage,
            offset: offset,
            q: this.search
          }
        }).then(response => {
          const list = response.data.data
          const l = list.length

          if(l > 0) {
            for (let i = 0; i < l; i++) {
              this.gif_list.push(list[i])
            }
            $state.loaded()
          } else {
            $state.complete()
          }

          if (this.gif_list.length === 0 && this.search) {

            this.notFoundGif();
          }
        }).catch(error => {
          console.log(error)
          $state.complete()
        })
      },
      reqUrl(){
        let url = `https://${this.host}/v1/gifs/trending`
        if(this.search && this.search !== ''){
          url = `https://${this.host}/v1/gifs/search`
        }
        return url
      },

      notFoundGif(){
        let url = `https://${this.host}/v1/gifs/random`
        axios.get(url, {
          params: {
            api_key: this.apiKey,
            tag: 'Not Found',
            rating: 'g'
          }
        })
        .then(response => {
          this.gif_list.push(response.data.data)
            $state.loaded()

        }).catch(error => {
          console.log(error)
          $state.complete()
        })
      }
    }
  }
</script>
