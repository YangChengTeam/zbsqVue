<template>
  <div id="list" ref="list">
   <div class="nav">
       <img src ="/static/image/back.png" @click="back"/>
       <h2 @click="back">{{title}}</h2>
   </div>
   <div  id="content2" class="minirefresh-wrap"  :style="'display:' + (isload?'block':'none')">
     <div class="minirefresh-scroll">    
       <div class="main-list">
        <div class="main-list-item" v-for="(item) in data" @click="goToDetail(item)">
             <img :src="item['front_img']" width="120" height="80">
             <div class="main-list-item-detail">
                 <div class="main-list-item-detail-title">
                    <h2>{{item['title']}}</h2>
                 </div>
                 <p>{{ item['desp'] }}</p>
                 <strong>使用<span>{{ item['build_num'] }}</span>次数</strong>
             </div>
        </div>
    </div>
  </div>
</div>
</div>
</template>

<script>
import MiniRefresh from "../../static/js/minirefresh.min.js"
import {Spinner} from 'spin.js';
export default {
  name: 'List',
  data () {
    return {
        id: 0,
        page: 1,
        data : [],
        miniRefresh: null, 
        type: '',
        isload: false,
        title: ""
    }
  },
  
  beforeRouteEnter (to, from, next) {
      next(vm=>{
           vm.id = to.params.id;
           vm.type = to.params.type;
           vm.title = to.params.title;
      });
  },
  mounted(){
      var that = this;
      this.miniRefresh = new MiniRefresh({
        container: '#content2',
        isScrollBar: false,
        down: {
           callback: function() {
            // 下拉事件
               that.page = 1;
               that.delete;
               that.data = [];
               that.loadData(that.type);
               that.miniRefresh.endDownLoading();
           }
       },
       up: {
          callback: function() {
             // 上拉事件
             that.loadData(that.type);
       }
    }
    });
  },
  methods : {
      back(){
       this.$router.go(-1);
     },
     loadData: function(type){
       this.type = type;
        var spinner ;
      if(this.page == 1){
             spinner = new Spinner(this.opts).spin(this.$refs.list);
      }
       this.$http.get("zbsq/index.php?m=Home&c=zbsq&a="+type+"&id=" + this.id + "&page=" + this.page).then(res => {
           if(spinner){
             spinner.stop();
           }
           var data = JSON.parse(res.bodyText);
           var list = data["data"];
           var end = false;
           if(list < 20){
              end = true;
           }
           if(list){
              this.data = this.data.concat(list);
           }
           this.page += 1;
           this.miniRefresh.endUpLoading(end);
           this.isload = true;
           }, err => {}
       );
     },
     goToDetail(item){
        this.$router.push({name:"Detail", params:{item}});
     }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
@import "../../static/css/minirefresh.min.css";

.main-list {
    margin-bottom: 10px;
    margin-top: 70px;
}

.main-list-item {
   display: flex;
   flex-direction: row;
   overflow: hidden;
   height: 100px;
   margin-left: 10px;
   margin-right: 10px;
   border: solid 1px #f1f1f1;
   border-radius: 2px; 
   box-shadow: 1px 3px 0px #aaa;
   margin-top: 10px;
   justify-content: flex-start;
   align-items: center;
   background: #fff;
}

/*.main-list-item:hover {
    background: rgba(0,0,0,0.2);
}*/

.main-list-item > img {
   border-radius: 3px; 
   margin-left: 10px;

}

.main-list-item-detail {
   display: flex;
   flex-direction: column;
   margin-left: 10px;
   height: 80px;
   justify-content: space-between;
   flex-grow: 2;
}

.main-list-item-detail  > p {
    font-size: 12px;
    color: #999;
    margin-right: 10px;
    line-height: 1.1;
}

.main-list-item-detail > strong  {
    font-size: 12px;
    color: #333;
}

.main-list-item-detail > strong > span  {
    font-size: 12px;
    color: #FF6347;
}

.main-list-item-detail-title {
   display: flex;
   flex-direction: row;
   justify-content: space-between;
}

.main-list-item-detail-title > h2 {
   font-weight: normal;
   font-size: 14px;
   color: #333;
   width: 80%;
   display: block;
}

.main-list-item-detail-title > strong {
   font-size: 12px;
   color: #ff8c00;
   display: inline-block;
   margin-right: 10px;
   text-align: right;
   width: 20%;
   display: block;
}
</style>
