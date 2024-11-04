<template>
  <div class="container">
    <button @click="startProjectionScreen">开启投屏</button>
    <button @click="sendMsg">发送消息</button>
    <!-- <button @click="closeLink">关闭连接</button> -->
    <button @click="endLink">结束调用</button>
  </div>
</template>

<script lang="ts">
// @ts-nocheck 不在当前文件中执行类型检查
import { toRefs, reactive, onMounted, watchEffect, onUnmounted } from 'vue'
export default {
  props: {},
  components: {},
  setup () {
    const presentationRequest = new PresentationRequest(['https://blog.fxnws.com']); // 投屏网址
    // const presentationRequest = new PresentationRequest(['test.html']); // 投屏本地页面
    navigator.presentation.defaultRequest = presentationRequest;
    let presentationConnection;
    // 传参
    const state = reactive({
    })
    const startProjectionScreen = () => {
      presentationRequest.start()
        .then(connection => {
          console.log('Connected to ' + connection.url + ', id: ' + connection.id);
        })
        .catch(error => {
          console.log(error);
        });
    }
    //发送消息
    const sendMsg = () => {
      const message = '发送消息测试';
      const lang = document.body.lang || 'en-US';

      console.log('Sending "' + message + '"...');
      presentationConnection.send(JSON.stringify({ message, lang }));
    }

    //关闭连接，关闭连接后无法再控制弹出窗口
    const closeLink = () => {
      console.log('Closing connection...');
      presentationConnection.close();
    }
    //关闭连接后，换可以重新再次连接，这里需要指定id
    // presentationRequest.reconnect(presentationId);

    //关闭弹窗，结束连接调用
    const endLink = () => {
      console.log('Terminating connection...');
      presentationConnection.terminate();
    }
    // 挂载
    onMounted(() => {
      //监视连接是否可用
      presentationRequest.addEventListener('connectionavailable', function (event) {
        presentationConnection = event.connection;
        presentationConnection.addEventListener('close', function () {
          console.log('> Connection closed.');
        });
        presentationConnection.addEventListener('terminate', function () {
          console.log('> Connection terminated.');
        });
        presentationConnection.addEventListener('message', function (event) {
          console.log('> ' + event.data);
        });
      });
      //监视可用的显示器
      presentationRequest.getAvailability()
        .then(availability => {
          console.log('Available presentation displays: ' + availability.value);
          availability.addEventListener('change', function () {
            console.log('> Available presentation displays: ' + availability.value);
          });
        })
        .catch(error => {
          console.log('Presentation availability not supported, ' + error.name + ': ' +
            error.message);
        });
    })
    // 监听
    watchEffect(() => {
      console.log('监听')
    })
    // 页面卸载
    onUnmounted(() => {
      console.log('页面卸载')
    })
    return {
      ...toRefs(state), endLink, closeLink, sendMsg, startProjectionScreen
    }
  }
}
</script>

<style scoped>
</style>
