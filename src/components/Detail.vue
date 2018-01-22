<template>
  <div id="detail" ref="detail">
       <div class="nav">
           <img src ="/static/image/back.png" @click="back" />
           <h2 @click="back">{{item.title}}</h2>
      </div>
       <div v-if ="isBuild" class="detail-main">
            <img :src="item['front_img']" />
            <div :class="item['is_hide'] == '1' ? 'detail-input-item-hide ' : ' ' + 'detail-input-item'" v-for="(item) in item['field']">
               <strong  :class="{'detail-input-item-hide' : true}">{{item['name']}}:</strong><input @change="processFile($event)" v-model="item['sval']" v-if="item['input_type'] != 1" :type="rtype(item['input_type'], item['is_hide'])" :class="'detail-input-item-input detail-input-item-'+rtype(item['input_type'], item['is_hide'])"  :placeholder="item['def_val']" />
               <div  v-else  class="detail-input-item-select" >
                <select class="detail-input-item-input" v-model="selected" @change="select(selected)">
                     <option v-for="(select, index) in item['select']" :value="select['opt_text']">{{select['opt_text']}}</option>
                </select>
                <input v-if="selectHide" v-model="selectedText" type="text" class="detail-input-item-input detail-input-item-select-input" :placeholder="item['def_val']" />
               </div>
            </div>
            <button @click="createImage">一键生成</button>
       </div>
       <div v-else class="detail-build">
        <a download :href="buildInfo.data">
            <img :src="buildInfo.data" width="100%">
        </a>
       </div>
  </div>
</template>

<script>
import {Spinner} from 'spin.js';
export default {
  name: 'Detail',
  data () {
    return {
        item: {},
        selected: "",
        selectedText: "",
        selectHide: false,
        isBuild: true,
        buildInfo:{},
        file: null,
        opts: null
    }
  },
  mounted(){
     if(localStorage.getItem("detail")){
        this.item = JSON.parse(localStorage.getItem("detail"));
     }

     this.opts = {
  lines: 13, // The number of lines to draw
  length: 38, // The length of each line
  width: 17, // The line thickness
  radius: 45, // The radius of the inner circle
  scale: 0.3, // Scales overall size of the spinner
  corners: 1, // Corner roundness (0..1)
  color: '#000', // CSS color or array of colors
  fadeColor: 'transparent', // CSS color or array of colors
  opacity: 0.6, // Opacity of the lines
  rotate: 0, // The rotation offset
  direction: 1, // 1: clockwise, -1: counterclockwise
  speed: 1, // Rounds per second
  trail: 60, // Afterglow percentage
  fps: 20, // Frames per second when using setTimeout() as a fallback in IE 9
  zIndex: 2e9, // The z-index (defaults to 2000000000)
  className: 'spinner', // The CSS class to assign to the spinner
  top: '50%', // Top position relative to parent
  left: '50%', // Left position relative to parent
  position: 'absolute' // Element positioning
};
   

  },
  beforeRouteEnter (to, from, next) {
      next(vm=>{
          vm.item  =  to.params.item;
          if(vm.item){
             localStorage.setItem("detail", JSON.stringify(vm.item));
          } else {
             vm.item = JSON.parse(localStorage.getItem("detail"));
          }
      });
  },
  methods : {
     back(){
       this.$router.go(-1);
     },
     rtype(type, hide){
        if(hide == 1) return "hidden";
        if(type == 0){
           return "text";
        } else if(type > 1){
           return "file";
        }
     },
     processFile(event){
        this.file = event.target.files[0];
     },
     createImage(){
        var field = this.item['field'];
        var requestData = "{";
        var img = "";
        for(var i=0; i<field.length; i++ ){
            var type = field[i]['input_type'];
            var is_hide = field[i]['is_hide'];
            if(is_hide == "1"){
                var value = field[i]["sval"] || "";
                requestData += "\""+i+"\":\"" + value + "\"";
            }
            else if(type == 0){
               var value = field[i]["sval"];
               if(!value){
                   alert(field[i]["def_val"]);
                    return;
               }
               requestData += "\""+i+"\":\"" + value + "\"";
            } else if(type==1){
              if(this.selected == '自定义文字'){
                 requestData += "\""+i+"\":\"" + this.selectedText + "\"";
              } else {
                 requestData += "\""+i+"\":\"" + this.selected + "\"";
              }
            } else if(type > 1){
                img = field[i]["sval"];
            }
            if(i < field.length-1){
                requestData +=",";
            }
        }
        requestData += "}";

        var that = this;

        var formData = new FormData();
        formData.append('requestData', requestData);

        formData.append('id', that.item["id"]);
        formData.append('mime', "863062030230011");
        if(this.file){
            formData.append('img', this.file, "u.jpg");
        }

        var spinner = new Spinner(this.opts).spin(this.$refs.detail);
        this.$http.post("zbsq/index.php?m=Home&c=Zbsq&a=start_zb", formData,{
        headers: {
            'Content-Type': 'multipart/form-data'
        }
    }).then(res => {
              spinner.stop();
              var data = JSON.parse(res.bodyText);
              if(data['errCode'] == 0){
                 that.buildInfo = data;
                 that.isBuild = false;
              }
             
         }, err => {
           alert(err);
        });
     },
     select(val){
        if(val == '自定义文字'){
          this.selectHide = true;
        } else {
          this.selectHide = false;
        }
     }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.detail-main {
   display: flex;
   flex-direction: column;
   align-items: center;
   margin-top: 70px;
}

.detail-main > img {
   display: block;
   width: 60%;
}

.detail-main > button {
   display: block;
   width: 80%;
   background: #FF8C00;
   height: 40px;
   border: none;
   box-shadow: 1px 3px 0px #aaa;
   margin-top: 20px;
   color: #fff;
   margin-bottom: 20px;
}

.detail-main > button:hover {
   position: fix;
   border: none;
   box-shadow: 1px 1px 0px #aaa;
}

.detail-input-item {
   display: flex;
   flex-direction: column;
   width: 100%;
   margin-top: 10px;
   align-items: center;
}

.detail-input-item-select {
     width: 100%;
     margin-top: 10px;
}

.detail-input-item-input {
   display: block;
   height: 40px;
   width: 80%;
   border: solid 1px #999;
   border-radius: 3px;
   padding-left: 5px;
   margin: 0 auto;
   
}

.detail-build {
   display: flex;
   align-items: center;
   flex-direction: column;
   justify-content: space-around;
   margin-top: 70px;
}

.detail-build > a {
   width: 80%;
}

.detail-input-item-select-input {
    margin-top: 10px;
}

.detail-input-item-hide {
   display: none;
}

.detail-main-loading {
   position: relative;
   background: #000;
   width: 100px;
   height: 100px;
}

</style>
