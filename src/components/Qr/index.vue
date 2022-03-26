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
      <div class="btn cancel" @click="onClear">清除缓存</div>
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
        // alert(JSON.stringify(res.urls))
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
      // 获取具有指定属性的所有标签页
      chrome.tabs.query({
        active: true,
        currentWindow: true
      }, function(tabs) {
        const tab = tabs[0];
        vm.urlSecond = tab.url || '';
        // 如果当前的地址不在已存地址中，则塞入 urls 的末尾
        let hasUrl = -1;
        vm.urls.forEach((item, index) => {
          if (item.urlFirst === vm.urlFirst && item.urlSecond === vm.urlSecond) {
            return hasUrl = index;
          }
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
      if (leftIndex >= 0) {
        this.urlFirst = this.urls[leftIndex].urlFirst;
        this.urlSecond = this.urls[leftIndex].urlSecond;
      }
    },
    // 向右边查找
    onRight() {
      const rightIndex = this.curIndex + 1;
      if (rightIndex < this.urls.length) {
        this.urlFirst = this.urls[rightIndex].urlFirst;
        this.urlSecond = this.urls[rightIndex].urlSecond;
      }
    },
    // 保存输入的当前地址
    onSave() {
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
      background-color: rgb(114, 161, 247);
      border: 1px solid rgb(69, 130, 244);
      border-radius: 2px;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    .cancel {
      background-color: rgb(208, 212, 220);
      border: 1px solid rgb(139, 141, 147);
    }
  }
}
</style>
