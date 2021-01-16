<template>
<v-sheet>
  <v-card class="pb-2">
    <v-card-title>ファイルをアップ</v-card-title>
    <v-card-actions>
      <input type="file" ref="upload" name="upfile[]" @change="upload($event)" webkitdirectory>
    </v-card-actions>
  </v-card>

  <v-carousel height="800"
    hide-delimiter-background
    show-arrows-on-hover
  >
    <v-carousel-item v-for="(file, i) in files" :key="i">
    <v-card dark height="900" class="px-8">
      <v-card-title>{{file["name"]}}</v-card-title>
      <v-card-text>
        <v-textarea :value="file['code']" outlined full-width height="700" placeholder="ここにコードが表示されます"></v-textarea>
      </v-card-text>
    </v-card>

    </v-carousel-item>
  </v-carousel>

<v-btn @click="download">ダウンロード</v-btn>
</v-sheet>
</template>

<script>

export default {
  data:()=>({
    app_name:"",
    files:[{name:"ここにタイトル"}],//{name:String,code:String}
  }),
  methods:{
    async upload(event){
      this.files.splice(0,1)
      const files = event.target.files
      this.app_name = files[0].webkitRelativePath.replace(/\/.+/,"")
      console.log(this.app_name)
      for(const file of files){
        const path = file.webkitRelativePath //String

        //不要なファイルをフィルタリング
        if(!path.includes(".vue"))continue;
        if(path.includes(".nuxt"))continue;
        if(path.includes("node_modules"))continue;
        console.log(path)

        var text;
        if(path.includes("/components")){
          await this.convertComponents(file)
        }else if(path.includes("/pages")){
          await this.convertPages(file)
        }
      }
      await this.createLayoutsIndex()
    },
    createLayoutsIndex(){
      let self = this
      return new Promise((resolve,reject)=>{
        const top_lines =['<!DOCTYPE html>','<html>','  <head>','    <base target="_top">','      <?!= include("config")?>','  <\/head>','  <body>']
        for(const file_obj of self.files){
          const file_name = file_obj.name.replace("/","").replace(".html","")
          const str = `    <?!= include("${file_obj.name})") ?>`
          top_lines.push(str)
        }
        const end_lines = [`  <div id="app"><\/div>`,`  <script>`,`    new Vue({`,`      vuetify: vuetify,`,
        `      render: h => h(mailer)`,`    }).$mount("#app")`,`  <\/script>`,`  <\/body>`,`<\/html>`]
        const html_lines= top_lines.concat(end_lines)
        const code = html_lines.join("\n")
        self.files.push({name:"layouts/index.html",code:code})
        resolve("OK")
      })
    },
    async convertPages(file){
      let self = this
      return new Promise(async function(resolve,reject){
        let {name,code} = await self.commonProcessComponentsAndPages(file)
        code = await self.removeImport(code)
        self.files.push({name:name,code:code})
        resolve("OK")
      })
    },
    async convertComponents(file){
      let self = this
      return new Promise(async function(resolve,reject){
        const result = await self.commonProcessComponentsAndPages(file)
        self.files.push(result)
        resolve("OK")
      })
    },
    commonProcessComponentsAndPages(file){
      let self = this
      return new Promise((resolve,reject)=>{
        const reader = new FileReader()
        const variable = file.name.replace(".vue","") //ファイル名から変数名を決める
        const name = file.webkitRelativePath.replace(self.app_name+"/","").replace(".vue",".html")
        reader.readAsText(file)
        reader.onload = function(e){
          let text = e.target.result
          text = text
              .replace(`<template>`,`<script type="text/x-template" id="${variable}" >`)
              .replace(`</`+`template>`,`</`+`script>`)
              .replace(`export default`,`const ${variable} =`)
          resolve({name:name,code:text})
        }
      })
    },
    removeImport(text){
      return new Promise((resolve,reject)=>{
        const t = text.replace(/^import.*/gm,"")
        resolve(t)
      })
    },
    download(){

    }
  }
}
</script>
