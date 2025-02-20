<template>
  <router-view _key="$route.path" />
</template>

<script>
import './scss/app.scss';
// import "@fortawesome/fontawesome-free/js/all";
import { get } from 'vuex-pathify'
import { remote, ipcRenderer, shell } from 'electron'
import url from 'url'

export default {
  name: 'App',

  data(){ return {
    versions: {
      app: require('./../../package.json').version,
      electron: process.versions.electron,
      electronWebpack: require('electron-webpack/package.json').version
    }
  }},

  computed: {
      style: get('app/getStyle'),
  },

  watch: {
      style(){
          this.checkColorStyle()
      }
  },

  created(){
      this.checkColorStyle()
  },

  mounted(){
      this.$store.dispatch('app/started')
      this.registerChannel()
  },

  errorCaptured(err, vm, info) {
    // err: error trace
    // vm: component in which error occured
    // info: Vue specific error information such as lifecycle hooks, events etc.
    console.log(err, vm, info)
    alert(err)

    // TODO: Perform any custom logic or log to server
    // return false to stop the propagation of errors further to parent or global error handler
  },

  methods:{
      registerChannel(){
          ipcRenderer.on('main', (event, data) => {
              console.log(event, data)
              this.$message({ type: 'info', message: data });
          })

          ipcRenderer.on('main-error', (event, data) => {
              this.$message({ type: 'error', message: data })
          })

          ipcRenderer.on('error', (event, data) => {
              console.log(event, data)
              alert(data)
          })
      },

      open(b){
          shell.openExternal(b)
      },

      openWithAutoclose(url){
          window.open(url, 'Download', 'width=200,height=30,backgroundColor=black,frame=false,hide=true')
      },

      sendServer(msg){
          ipcRenderer.send('server', msg)
      },

      openServer(){
          ipcRenderer.send('server-show')
      },

      show(data){
          ipcRenderer.send('show', data)
      },

      sendMain(msg){
          console.log('sending to main', msg)
          ipcRenderer.send('main', msg)
      },

      sendPS4(msg){
          console.log('sending to ps4', msg)
          ipcRenderer.send('ps4', msg)
      },

      notify(data){
          ipcRenderer.send('notify', data)
      },

      log(msg='', data={}, type='log'){
          this.$root.sendPS4({ time: Date.now(), msg, data, type })
      },

      getImage(img){
          const isDevelopment = process.env.NODE_ENV === 'development';
          // const staticPath = isDevelopment ? __static : __dirname.replace(/app\.asar$/, 'static');

          if(isDevelopment)
            return url.resolve(window.location.origin, img);

          return __dirname.replace(/app\.asar$/, 'static') + '/' + img
      },

      checkColorStyle(){
          document.getElementsByTagName('html')[0].classList.remove('dark')
          document.getElementsByTagName('html')[0].classList.remove('light')
          document.getElementsByTagName('html')[0].classList.remove('pureblack')
          document.getElementsByTagName('html')[0].classList.add(this.style)
      }

  },
}
</script>
<style lang="css" scoped>
</style>
