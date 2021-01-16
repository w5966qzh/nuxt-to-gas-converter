<template>
<v-sheet>
  <v-card>ファイルをアップ
    <input type="file" ref="upload" name="upfile[]" @change="upload($event)" webkitdirectory>
  </v-card>

  <v-carousel
    height="400"
    hide-delimiter-background
    show-arrows-on-hover
  >
    <v-carousel-item v-for="(text, i) in texts" :key="i">
    <v-card dark>
      <v-card-title></v-card-title>
      <v-card-text>{{text}}</v-card-text>
      <v-card-text>
        <v-textarea :value="text" outlined></v-textarea>
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
    files:{},
    texts:[],
    test:"some"
  }),
  methods:{
    async upload(event){
      const files = event.target.files
      for(const file of files){
        const path = file.webkitRelativePath;
        //不要なファイルをフィルタリング
        if(!path.includes(".vue"))continue;
        if(path.includes(".nuxt"))continue;
        if(path.includes("node_modules"))continue;
        console.log(path)

        var text;
        if(path.includes("/components")){
            console.log(`${path}はcomponentsで処理`);
            text = await this.convertComponents(file)
            this.texts.push(text)
        }else if(path.includes("/pages")){
            console.log(`${path}はpagesで処理`);
            text = await this.convertComponents(file)
            text = await this.removeImport(text)
            this.texts.push(text)
        }else if(path.includes("/layouts")){

        }
      }
    },
    removeImport(text){
      return new Promise((resolve,reject)=>{
        const t = text.replace(/^import.*/,"")
        resolve(t)
      })
    },
    convertComponents(file){
      return new Promise((resolve,reject)=>{
        const reader = new FileReader()
        const variable = file.name.replace(".vue","") //ファイル名から変数名を決める
        reader.readAsText(file)
        reader.onload = function(e){
          let text = e.target.result;
          text = text.replace(`<template>`,`<script type="text/x-template" id="${variable}" >`);
          text = text.replace(`</`+`template>`,`</`+`script>`)
          text = text.replace(`export default`,`const ${variable} =`)
          resolve(text)
        }
      })
    }

  }
}
</script>
