<template>
  <div id="app">
    <transition name="fade">
      <router-view></router-view>
    </transition>
  </div>
</template>

<script>
  const { ipcRenderer } = require('electron');
  const { BrowserWindow } = require('electron').remote;

  export default {
    name: 'electronTemplate',
    computed: {
      // ...mapState({ path: state => state.global.path }),
    },
    beforeMount() {
      window.addEventListener('offline', (e) => {
        alert('网络断开连接！');
        ipcRenderer.send('WIFIoffline');
      });
      window.addEventListener('online', (e) => {
        alert('网络已连接');
        ipcRenderer.send('WIFIonline');
      });
      if (navigator.onLine) {
        ipcRenderer.send('WIFIonline');
      }
      // 实时数据获取
      ipcRenderer.on('realTime', (event, data) => {
        this.$store.dispatch(`PLC${data.id}Data`, data.data);
      });
      // 实时数据获取X
      ipcRenderer.on('realTime550', (event, data) => {
        const b = data.data[0] === '1';
        // console.log(`PLC${data.id}550`, b);
        this.$store.dispatch(`PLC${data.id}550`, b);
      });
      // 实时数据获取S
      ipcRenderer.on('realTimeS', (event, data) => {
        this.$store.dispatch(`PLC${data.id}S`, data.data);
      });
      // 连接成功
      ipcRenderer.on('lineOK', (event, data) => {
        this.$notify.success({
          showClose: true,
          duration: 0,
          title: '成功',
          message: `PLC${data.id}${data.data}`,
        });
        this.$store.dispatch(`PLC${data.id}State`, true);
      });
      // 连接中断
      ipcRenderer.on('lineError', (event, data) => {
        this.$store.dispatch(`PLC${data.id}State`, false);
        this.$notify.error({
          showClose: true,
          duration: 5000,
          title: '错误',
          message: `PLC${data.id}${data.data}`,
        });
        // ipcRenderer.send('mudbusError', data.id);
      });
    },
    watch: {
    },
    methods: {
    },
  };
</script>

<style lang="scss">
@import "./css/elementUI.scss";
@import "./css/global.scss";
@import "./css/menuTemplate.scss";
@import "./css/deviceInfo.scss";
@import "./css/frames.scss";
// @import "./css/animate.css";
// @import "./css/task_record.scss";
// 路由切换过度动画
.fade-enter-active, .fade-leave-active {
  transition: opacity .1s
}
.fade-enter, .fade-leave-to /* .fade-leave-active in below version 2.1.8 */ {
  opacity: 0
}
</style>
