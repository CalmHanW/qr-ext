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
        <input
          style="flex:1"
          type="text"
          name=""
          :disabled="!isCheck"
          v-model="urlFirst"
        >
        <input class="checkbox" type="checkbox" name="" v-model="isCheck">
      </div>
      <div class="input layout">
        <input style="flex:1" type="text" name="" v-model="urlSecond">
        <input class="checkbox" type="checkbox" name="" v-model="isEncode">
      </div>
    </div>
    <div class="btn-con">
      <div class="btn" @click="onSave">保存</div>
      <div class="btn" @click="onClear">清除缓存</div>
    </div>
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
      isCheck: false, // 是否有拼接的前缀
      isEncode: false, // url 是否编码
      urlFirst: '', // 前部分拼接的 url
      urlSecond: '', // 后部分拼接的 url（主url）
      urls: [], // 存储的链接数组集
    };
  },

  computed: {
    url: {
      get() {
        return this.urlFirst + (this.isEncode ? encodeURIComponent(this.urlSecond) : this.urlSecond);
      },
      // set(value) {
      //   this.urlSecond = value;
      // },
    },
    curIndex() {
      let index = -1;
      this.urls.forEach((item, k) => {
        if (item.urlFirst === this.urlFirst && item.urlSecond === this.urlSecond) {
          index = k;
        }
      })
      return index;
    }
  },

  mounted() {
    const vm = this;
    // 当纯 HTML被完全加载以及解析时，DOMContentLoaded 事件会被触发，而不必等待样式表，图片或者子框架完成加载
    document.addEventListener('DOMContentLoaded', function() {
      // 获取已存储的 url
      chrome.storage.local.get("urls", (res) => {
        alert(JSON.stringify(res.urls))
        // alert(res.urls.length)
        if(res.urls) {
          vm.urls.splice(0, 0, ...res.urls);
        }
        vm.getCurrentTab();
      });
    });
  },

  methods: {
    getCurrentTab() {
      const vm = this;
      chrome.tabs.query({
        active: true,
        currentWindow: true
      }, function(tabs) {
        const tab = tabs[0];
        vm.urlSecond = tab.url || '';

        // const index = vm.urls.indexOf(vm.urlSecond);
        // alert(JSON.stringify(vm.urls));
        let hasUrl = -1;
        vm.urls.forEach((item, index) => {
          alert(vm.urlFirst);
          if (item.urlFirst === vm.urlFirst && item.urlSecond === vm.urlSecond) {
            hasUrl = index;
          }
          // return item.urlFirst === vm.urlFirst && item.urlSecond === vm.urlSecond
        })

        if (hasUrl === -1) {
          vm.urls.push({
            urlFirst: vm.urlFirst,
            urlSecond: vm.urlSecond,
            isStorge: false,
          });

          // // 先将当前页面进行存储(当前页面自动保存)
          // chrome.storage.local.set({
          //   urls: vm.urls
          // });
        }
      });
    },
    // 向左边查找
    onLeft() {
      const leftIndex = this.curIndex - 1;
      alert(leftIndex);
      if (leftIndex >= 0) {
        // this.url = this.urls[leftIndex];
        this.urlFirst = this.urls[leftIndex].urlFirst;
        this.urlSecond = this.urls[leftIndex].urlSecond;
      }
    },
    // 向右边查找
    onRight() {
      const rightIndex = this.curIndex + 1;
      alert(rightIndex);
      if (rightIndex < this.urls.length) {
        // this.url = this.urls[rightIndex];
        this.urlFirst = this.urls[rightIndex].urlFirst;
        this.urlSecond = this.urls[rightIndex].urlSecond;
      }
    },
    // 保存当前地址
    onSave() {
      // 是否已经存储
      // let isStorge = false;
      // this.urls.forEach((item) => {
      //   if (item === this.url) {
      //     isStorge = true;
      //   }
      // })

      if (this.curIndex === -1 || !this.urls[this.curIndex].isStorge) {
        this.urls.push({
          urlFirst: this.urlFirst,
          urlSecond: this.urlSecond,
          isStorge: true,
        });
        chrome.storage.local.set({
          urls: this.urls
        });
      }
    },
    // 清空所有地址
    onClear() {
      chrome.storage.local.clear();
    },
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
  .btn-con {
    display: flex;
    justify-content: space-around;

    .btn {
      margin-top: 16px;
      height: 30px;
      width: 120px;
      background-color: rgb(148, 175, 224);
      border-radius: 2px;
      display: flex;
      justify-content: center;
      align-items: center;
    }
  }
}
</style>
