<template>
  <div id="main" ref="main">
    <div class="nav">
      <img src="/static/image/logo.png" height="25px" />
   </div>
   <div  id="content" class="minirefresh-wrap" :style="'display:' + (isload?'block':'none')">
     <div class="minirefresh-scroll" >        
   
  <div class="swiper-container" >
    <div class="swiper-wrapper">

      <div class="swiper-slide" v-for="(item) in banner" @click="goToList(item['id'], 'getSlideMore',item['title'])">
          <img :src="item['c_img']" width="100%" height="100%" />
      </div>
      
    </div>
    <!-- Add Pagination -->
    <div class="swiper-pagination"></div>

    </div>

    <div class="main-menu">
        <div :class="{'main-menu-item':true, 'main-menu-item-rb' : (index + 1) % 3 == 0}" v-for="(item, index) in channel" @click="goToList(item['id'], 'getCateList', item['name'])">
             <img :src="item['ico']" width="40" />
             <strong>{{item['name']}}</strong>
        </div>
    </div>

    <div class="main-list">
        <div class="main-list-item" v-for="(item) in data" @click="goToDetail(item)">
             <img :src="item['front_img']" width="120" height="80">
             <div class="main-list-item-detail">
                 <div class="main-list-item-detail-title">
                    <h2>{{item['title']}}</h2>
                    <strong>{{ channel[item['type']]['name'] }}</strong>
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
import Swiper from "../../static/js/swiper.min.js"
import MiniRefresh from "../../static/js/minirefresh.min.js"
import {Spinner} from 'spin.js';

export default {
  name: 'Main',
  data () {
    return {
       banner: [],
       channel: [],
       data: [],
       page: 1,
       miniRefresh: null,
       isload: false,
       opts: null
    }
  },

  mounted(){
    var that = this;
    var swiper = new Swiper('.swiper-container', {
      pagination: {
        el: '.swiper-pagination',
        clickable: true
      },
    });

    this.loadData();
    this.miniRefresh = new MiniRefresh({
    container: '#content',
    isScrollBar: false,
    down: {
        callback: function() {
            // 下拉事件
            that.page = 1;
            that.loadData();
            that.miniRefresh.endDownLoading();
        }
    },
    up: {

        callback: function() {
            // 上拉事件
            that.loadData();
            
        }
    }
    });
  },
  methods : {
     loadData: function(){
      var spinner ;
      if(this.page == 1){
          spinner = new Spinner(this.opts).spin(this.$refs.main);
      }
       this.$http.get("zbsq/index.php?m=Home&c=Zbsq&a=test1&version=3.1.5&page=" + this.page).then(res => {
         var data = JSON.parse(res.bodyText);
         this.banner = data["banner"];
         this.channel = data["channel"];
         var list = data["data"];
         var end = false;
         if(list < 20){
             end = true;
         }
         if(list){
             this.data = this.data.concat(list);
         }
         if(spinner){
            spinner.stop();
        }
         this.page += 1;
         this.miniRefresh.endUpLoading(end);
         this.isload = true;
     }, err => {
    });
     },
     goToList(id, type, title){
        this.$router.push({name:"List", params:{id, type, title}});
     },
     goToDetail(item){
        this.$router.push({name:"Detail", params:{item}});
     }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
  @import "../../static/css/swiper.min.css";


  .swiper-container {
      width: 100%;
      height: 150px;
      overflow: hidden;
}
.swiper-slide {
     
}
.swiper-pagination-bullet {
      width: 10px;
      height: 10px;
      text-align: center;
      line-height: 10px;
      font-size: 12px;
      color:#000;
      opacity: 1;
      background: #fff;
}
.swiper-pagination-bullet-active {
      color:#fff;
      background: rgba(0,0,0,0.4);
}
</style>
<style scoped>
#content {
   margin-top: 60px;
}


.main-menu {
   display: flex;
   flex-direction: row;
   flex-wrap: wrap;
   background: #fff;
}

.main-menu-item {
   display: flex;
   width: 33.05%;
   height: 100px;
   flex-direction: column;
   justify-content: center;
   border-bottom: solid 1px #f1f1f1;
   border-right: solid 1px #f1f1f1;
}

/*.main-menu-item:hover {
   background: rgba(0,0,0,0.2);
}*/

.main-menu-item > img {
  display: block;
  margin: 0 auto;
}

.main-menu-item > strong {
   text-align: center;
   font-size: 12px;
   font-weight: normal;
   color: #000;
}

.main-menu-item-rb {
   border-right: none;
   flex-grow: 2;
}

.main-list {
    margin-bottom: 10px;
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
