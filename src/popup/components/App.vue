<template>
  <div class="pop-container">
    <el-tabs v-model="activeName">
      <el-tab-pane label="环境管理" name="env">
        <el-row v-for="item in engines" :key="item.id">
          <el-col :span="24">
            <el-button type="text" @click="search(item)" :style="randomRgb()">
              {{ item.id }}
            </el-button>
            <el-divider direction="vertical"></el-divider>
            <el-button type="text" @click="search(item)" :style="randomRgb()">
              {{ item.name }}
            </el-button>
            <el-divider direction="vertical"></el-divider>
            <el-button type="text" @click="search(item)" :style="randomRgb()">
              {{ item.url }}
            </el-button>
          </el-col>
        </el-row>
      </el-tab-pane>
      <el-tab-pane label="滑词翻译" name="translate">
        <div class="switch-wrapper">
          <div class="switch-desc">是否启用划词翻译</div>
          <input
            type="checkbox"
            class="switch"
            :checked="switchChecked"
            @click="addSwitchListen"
          />
        </div>
      </el-tab-pane>
      <el-tab-pane label="角色管理" name="third">角色管理</el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
export default {
  name: "App",

  components: {},

  data() {
    return {
      engines: [],
      activeName: "env",
      switchChecked: false,
    };
  },

  methods: {
    randomRgb() {
      let R = Math.floor(Math.random() * 255);
      let G = Math.floor(Math.random() * 255);
      let B = Math.floor(Math.random() * 255);
      return {
        color: "rgb(" + R + "," + G + "," + B + ")",
      };
    },
    init() {
      let defaultConfig = {
        engines:
          '[{"name":"决策平台7220环境","url":"http://10.100.1.13:7220/ss/#/","inPopup":true,"id":"1","inShortcuts":true,"inRight":true},{"name":"决策平台7221环境","url":"http://10.100.1.13:7221/ss/#/","inPopup":true,"id":"2","inRight":false,"inShortcuts":true}]',
      }; // 默认配置
      // 读取数据，第一个参数是指定要读取的key以及设置默认值
      chrome.storage.sync.get(defaultConfig, items => {
        this.engines = JSON.parse(items.engines);
        this.engines = this.engines.filter(item => {
          if (item.name && item.url && item.inPopup === true) {
            return item;
          }
        });
      });
      chrome.storage.sync.get(["checked"], target => {
        if (target) {
          this.switchChecked = target.checked;
        }
      });
    },
    search(item) {
      chrome.tabs.getSelected(null, function(tab) {
        let urlObj = parseUrl(tab.url);
        let query =
          urlObj["wd"] ||
          urlObj["word"] ||
          urlObj["w"] ||
          urlObj["q"] ||
          urlObj["query"];
        if (query) {
          chrome.tabs.update({
            url: item.url.replace("%s", query),
          });
          window.close();
        } else {
          window.open(item.url.replace("%s", ""));
          window.close();
        }
      });
    },
    addSwitchListen(e) {
      chrome.storage.sync.set({ checked: e.target.checked });

      chrome.tabs.query({ active: true, currentWindow: true }, tabs => {
        chrome.tabs.sendMessage(tabs[0].id, {
          checked: e.target.checked,
          type: "CHECKED",
        });
      });
    },
  },

  created() {
    this.init();
  },

  computed: {},

  mounted() {},
};

function parseUrl(url) {
  let obj = {};
  let reg = /([^?=&]+)=([^?=&]+)/g;
  url.replace(reg, function() {
    obj[arguments[1]] = arguments[2];
  });
  return obj;
}
</script>
<style lang="css">
@import "./index.css";
</style>
