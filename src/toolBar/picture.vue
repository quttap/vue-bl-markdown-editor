<!--图片-->

<template>
    <div class="picture">
      <div class="mark-down-name" @mouseenter="close=false">
        <i class="fa fa-photo " title="图片"></i>
        <div class="tool-bar-box mark-down-form-box" :style="close?'display: none;':''">
          <div>
            <input v-model="name" type="text" placeholder="请输入图片名字">
          </div>
          <div>
            <input v-model="url" type="text" placeholder="请输入图片地址">
          </div>
          <div>
            <input type="file" placeholder="上传图片" @change="changeFile">
          </div>
          <div>
            <button style="width: 100%;" @click="onClick(name,url)">确认</button>
          </div>
          <div style="display: flex;flex-wrap: wrap;">
            <div v-for="(pic,index) in picList" :key="index" title="点击删除" @click="clearPic(pic)">
              <img  style="width: 100%;max-width: 100px;" :src="pic" alt="">
            </div>
          </div>
        </div>
      </div>
    </div>
</template>

<script type="text/ecmascript-6">
    export default {
        name: "picture",
      data() {
        return {
          //动态自动注入
          parent: null,
          close: false,

          name: null,
          url: null,

          config: null,
          picList: []
        }
      },
      watch:{
          'parent.content'(val){
            //解析图片
            this.resolvingPicList(val);

          }
      },
      created(){
        this.config = this.parent.config.picture;
          //监听粘贴，拖拽图片
        document.addEventListener('paste', e=>{
          this.onPaste(e)
        })
        document.addEventListener('drop', e=>{
          this.onDrag(e)
        })

        this.resolvingPicList(this.parent.content);
      },
      methods: {
        onClick(name,url) {
          this.close = true;
          this.parent.insertContent('![',name,']('+url+')');
        },
        //粘贴事件
        onPaste(event){
          console.log('onPaste',event);
          if (event.clipboardData || event.originalEvent) {
            //某些chrome版本使用的是event.originalEvent
            let clipboardData = (event.clipboardData || event.originalEvent.clipboardData);
            let file = this.getFiles(clipboardData);
            if (file==null){
              return;
            }
            console.log('onPaste',file);
            this.uploadFile(file,'paste');
          }
        },
        //拖拽事件
        onDrag(event){
          //拦截其他事件
          console.log('onDrag',event);
          event.preventDefault();
          let clipboardData = event.dataTransfer;
          let file = this.getFiles(clipboardData);
          console.log('onDrag',file);
          this.uploadFile(file,'drag');

        },
        changeFile(event){
          let file = event.target.files.length>0 && event.target.files[0];
          this.uploadFile(file,'upload');
        },
        getFiles(dataTransfer){
          if(dataTransfer.items){
            // for chrome
            let  items = dataTransfer.items;
            if (!items) return;
            let types = dataTransfer.types || [];
            let files = [];
            let item = null;
            for (var i = 0; i < types.length; i++) {
              if (types[i] === 'Files') {
                files.push(items[i]);
                item = items[i];
              }
            }
            if (item && item.kind === 'file') {
              event.preventDefault();
              let file = item.getAsFile();
              return file;
            }
          }
        },

        uploadFile(file,from){
          this.config.uploadCallback(file,from).then(data=>{
            console.log('上传完毕回调',data);
            this.onClick(data.name,data.url);
          });
        },



        //解析内容获取图片列表,发现 貌似语音也是用的这个解析
        resolvingPicList(val){
          if (this.config!=null && this.config.resolving === false) return;
          let parses = this.parent.md.parse(val, {
            references: {}
          });
          let picListSet = new Set();
          parses.forEach(parse=>{
            if (parse.type === "inline" && parse.children!=null){
              parse.children.forEach(img=>{
                if (img.tag === "img"){
                  picListSet.add(img.attrs[0][1]);
                }
              })
            }
          })
          this.picList = [...picListSet];
        },
        clearPic(pic){
          let content = this.parent.content;
          let reg = new RegExp('\\!\\[(.*?)\\]\\('+pic+'\\)','g');
          content = content.replace(reg,'');
          this.parent.content = content;
        }
      },
    }
</script>

<style>
.picture {

}
</style>
