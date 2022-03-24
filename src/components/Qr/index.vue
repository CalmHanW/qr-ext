<template>
  <div class="qr">
    <div class="qr-top">
      <img
        class="arrow"
        src="@/assets/arrow.png" alt=""
        @click="onLeft"
      >
      <vueQr
        :correctLevel="3"
        :autoColor="false"
        :text="url"
        :size="200"
        :margin="0"
        :logoMargin="3"
      ></vueQr>
      <img
        class="arrow arrow-right"
        src="@/assets/arrow.png" alt=""
        @click="onRight"
      >
    </div>
    <div class="form">
      <textarea class="url" v-model="url" readonly :rows="3"></textarea>
      <div class="input layout">
        <input style="flex:1" type="text" name="" v-model="urlFirst">
        <input class="checkbox" type="checkbox" name="" v-model="isCheck">
      </div>
      <input class="input" type="text" name="" v-model="urlSecond">
    </div>
    <div class="btn" @click="onSave">保存</div>
  </div>
</template>

<script>
import vueQr from 'vue-qr'
export default {
  name: 'Qr',

  components: {
    vueQr,
  },

  data() {
    return {
      url: '',
      isCheck: false,
      urlFirst: '',
      urlSecond: '',
      urls: [],
    };
  },

  mounted() {
    const vm = this;
    document.addEventListener('DOMContentLoaded', function() {
      const queryInfo = {
        active: true,
        currentWindow: true
      };

      chrome.tabs.query(queryInfo, function(tabs) {
        const tab = tabs[0];
        vm.url = tab.url || '';
        vm.urls.push(vm.url)
      });
    });

    // 获取已存储的 url
    chrome.storage.local.get("urls", (res) => {
      if(res.urls) {
        vm.urls.splice(0, 0, ...res.urls);
      }
    });
  },

  methods: {
    onLeft() {
      // alert('onLeft')
    },
    onRight() {
      // alert('onRight')
    },
    onSave() {
      chrome.storage.local.set(this.urls);
    }
  },
};
</script>

<style lang="scss" scoped>
.qr {
  .qr-top {
    display: flex;
    justify-content: center;
    align-items: center;

    .arrow {
      height: 24px;
      width: 12px;
      padding-right: 24px;
    }

    .arrow-right {
      transform: rotateZ(180deg);
    }

    .arrow:hover {
      cursor: pointer;
    }
  }
  .form {
    width: 100%;
    margin-top: 24px;
    color: black;
    .url {
      width: 100%;
    }
    .input {
      width: 100%;
      margin-top: 16px;
      display: block;
    }
    .layout {
      display: flex;
    }
  }
  .btn {
    margin-top: 16px;
    height: 30px;
    width: 100%;
    background-color: rgb(148, 175, 224);
    border-radius: 2px;
    display: flex;
    justify-content: center;
    align-items: center;
  }
}
</style>
