<template>
  <v-layout class="rounded rounded-md fill-height">
    <v-navigation-drawer
      expand-on-hover
      rail
    >
      <v-list>
        <v-list-item
          prepend-avatar="https://cdn.pixabay.com/photo/2021/11/12/03/04/woman-6787784_1280.png"
          subtitle="pingtrans@gmailcom"
          title="Ping's"
        ></v-list-item>
      </v-list>
      <v-divider></v-divider>

      <v-list density="compact" nav>
        <v-list-item prepend-icon="mdi-folder" title="My Files" value="myfiles"></v-list-item>
        <v-list-item prepend-icon="mdi-account-multiple" title="Shared" value="shared"></v-list-item>
        <v-list-item prepend-icon="mdi-star" title="Starred" value="starred"></v-list-item>
      </v-list>
    </v-navigation-drawer>
    <v-main class="d-flex">


      <!--      <v-btn :onclick="deepLRequest">request test</v-btn>-->

      <v-container>
        <v-responsive
          class="align-centerfill-height mx-auto"
          max-width="90%"
        >
          <div class="py-4"/>
          <v-row>
            <v-col cols="4">
              <v-combobox
                chips
                label="翻译语言"
                v-model="select"
                :items="supportLanguage"
                variant="outlined"
                density="compact"
              ></v-combobox>
            </v-col>
            <!--            <v-col cols="2">-->
            <!--              <v-file-input-->
            <!--                clearable-->
            <!--                variant="outlined"-->
            <!--                density="compact"-->
            <!--                label="批量翻译"-->
            <!--              ></v-file-input>-->
            <!--            </v-col>-->

            <!--            <v-col cols="4">-->
            <!--              <v-btn-->
            <!--                id="menu-activator"-->
            <!--                icon="mdi-cog"-->
            <!--                size="small"-->
            <!--              ></v-btn>-->
            <!--              <v-menu activator="#menu-activator"-->
            <!--                      location="end">-->
            <!--                <v-list>-->
            <!--                  <v-list-item-->
            <!--                    v-for="(item, index) in items"-->
            <!--                    :key="index"-->
            <!--                    :value="index"-->
            <!--                  >-->
            <!--                    <v-list-item-title>{{ item.title }}</v-list-item-title>-->
            <!--                  </v-list-item>-->
            <!--                </v-list>-->
            <!--              </v-menu>-->
            <!--            </v-col>-->


            <v-col cols="10" style="margin-top: -30px">
              <v-textarea label="翻译文本" variant="outlined" v-model="text"></v-textarea>
            </v-col>

            <v-col cols="4">
              <v-combobox
                v-model="target"
                :items="supportLanguage"
                label="目标语言"
                variant="outlined"
                density="compact"
                chips
                multiple
              ></v-combobox>

            </v-col>
            <v-col cols="8">
              <v-btn @click="translate" style="margin-right: 10px">翻译</v-btn>
              <v-btn @click="exportExcel">导出 Excel</v-btn>
            </v-col>
            <v-col cols="4"
                   v-for="(item, index) in showResult"
                   :key="index"
            >
              <v-card
                :title="item.title"
              >
                <v-card-text
                  v-for="(content, index) in item.content"
                  :key="index"
                  :class="{ 'pb-0': index !== item.content.length - 1 }"
                >
                  {{ content }}
                </v-card-text>
              </v-card>
            </v-col>
          </v-row>
        </v-responsive>
      </v-container>


    </v-main>
  </v-layout>
</template>

<script>
import axios from "axios";
import {TranslatorsResult} from "@/class/Moudles";
import * as XLSX from "xlsx";


const apiKey = '';
// 创建一个axios实例，可以设置默认的配置
const request = axios.create({
  timeout: 30000,  // 设置超时时间
  headers: {
    'Content-Type': 'application/json',
    'Access-Control-Allow-Origin': '*',
    'Authorization': apiKey,
    // 可以在这里添加其他需要的头部配置
  },
});


const supportLangMap = {
  '🇨🇳 汉语': 'ZH',
  '🇬🇧 英语': 'EN',
  '🇩🇪 德语': 'DE',
  '🇷🇺 俄语': 'RU',
  '🇫🇷 法语': 'FR',
  '🇵🇹 葡萄牙': 'PT',
  '🇪🇸 西班牙': 'ES',
  '🇯🇵 日语': 'JA',
  '🇹🇼 繁体中文': 'ZH-HANT',
  '🇮🇹 意大利': 'IT',
}

export default {
  data: () => ({
    items: [
      {title: 'Click Me'},
      {title: 'Click Me'},
      {title: 'Click Me'},
      {title: 'Click Me 2'},
    ],
    supportLanguage: Object.keys(supportLangMap),
    select: null,
    text: null,
    // 选择的语言
    target: [],
    request: null,
    desserts: [
      {}
    ],
    showResult: [],

    rows: [
      {name: "George Washington", birthday: "1732-02-22"},
      {name: "John Adams", birthday: "1735-10-19"}
    ]

  }),

  onMounted() {
    console.log("mounted")

  },
  methods: {
    exportExcel() {
      console.log("exportExcel function")
      console.log(this.showResult)

      if (this.showResult === null) {
        return
      }
      // 创建工作簿和工作表
      const wb = XLSX.utils.book_new();
      // const ws_data = [
      //   ["Name", "Age", "Location"],
      //   ["Alice", 25, "New York"],
      //   ["Bob", 30, "Los Angeles"]
      // ];
      const ws_data = []

      console.log(JSON.stringify(this.showResult))

      // [{"title":"原文","content":["你好","同类"]},{"title":"🇬🇧 英语","content":["How are you?","same type"]},{"title":"🇷🇺 俄语","content":["Как дела?","один тип"]}]
      const title = this.showResult.map(item => item.title)
      ws_data.push(title)

      const maxContentLength = Math.max(...this.showResult.map(item => item.content.length));
      for (let i = 0; i < maxContentLength; i++) {
        const row = this.showResult.map(item => item.content[i] || ""); // 用空字符串填充缺失的内容
        ws_data.push(row);
      }

      const ws = XLSX.utils.aoa_to_sheet(ws_data);
      XLSX.utils.book_append_sheet(wb, ws, "Sheet1");

      // 生成Excel文件并触发下载
      XLSX.writeFile(wb, "ExampleData.xlsx");
    },
    // 调用翻译
    translate() {
      console.log("translate")
      let text = this.text;
      console.log(text)
      // 待翻译文本
      if (text === null || text === '' || this.target.length === 0) {
        return
      }

      this.showResult = []

      let tranText = text.split('\n')
      // 展示原文本
      this.showResult.push(new TranslatorsResult('原文', tranText))
      console.log("this.showResult")
      console.log(this.showResult)
      console.log('选择的语言:' + this.target)
      this.target = this.target.filter(item => item !== this.select)

      let requests = this.target.map((lang) => {
        return new Promise((resolve, reject) => {
          console.log('目标语言:' + lang);
          this.deepLRequest(tranText, this.select, lang, resolve, reject);
        });
      });

      Promise.all(requests)
        .then(() => {
          console.log(this.showResult)
          console.log(this.target)
        })
        .catch((error) => {
          console.error("一个或多个请求失败:", error);
        });
    },

    deepLRequest(tranText, sourceLang, targetLang, resolve, reject) {
      console.log("API test");
      request.post('/api/translate', {
        text: tranText,
        source_lang: supportLangMap[sourceLang],
        target_lang: supportLangMap[targetLang]
      })
        .then(({data}) => {
          console.log(data);
          let contents = [];
          data.translations.forEach((item) => {
            console.log(item.text);
            contents.push(item.text);
          });
          this.showResult.push(new TranslatorsResult(targetLang, contents));
          resolve();
        })
        .catch((error) => {
          console.error(error);
          reject(error);
        });
    }
  }
}
</script>

<style scoped>
</style>
