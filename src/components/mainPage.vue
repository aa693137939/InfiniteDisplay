<template>
<div class="content">
  <header>
    <ul class="tab-nav" v-if="tabList.length<=5">
      <li class="nav-li" v-for="item in tabList" :title="item.name"
          :class="{true:'active'}[selTabId==item.tab_id]" @click="toSelTab(item)">
        {{ item.name }}
      </li>
    </ul>
    <ul class="tab-nav more-tab" v-if="tabList.length>5">
      <span class="tab-more tab-more-L" @click="toMoreTab('L')" v-if="smallFlag>0">←</span>
      <li class="nav-li" v-for="item in tabSmallList[smallFlag]" :title="item.name"
          :class="{true:'active'}[selTabId==item.tab_id]" @click="toSelTab(item)">
        {{ item.name }}
      </li>
      <span class="tab-more tab-more-R" @click="toMoreTab('R')"
            v-if="smallFlag<Math.ceil(tabList.length/5)-1">→</span>
    </ul>
  </header>
  <img class="content-bg" :src="bgImage">
  <div v-infinite-scroll="getTabContent" infinite-scroll-disabled="busy" infinite-scroll-distance="100" class="clearfix">
    <ul class="content-ul">
      <li v-for="item in contentData" v-if="item.template" :style="item.template.liStyle">
        <p :style="item.template.titleStyle">{{ item.content.resData.title_name }}</p>
        <div v-for="tempPoster in item.template.posters">
          <div class="poster-div" v-for="contentPoster in item.content.postList"
               v-if="tempPoster.posterData.poster_site_id==contentPoster.data.poster_site_id"
               :style="tempPoster.divStyle" :class="{'posterType0':tempPoster.layout_style==0,
                  'posterType1':tempPoster.layout_style==1,
                  'posterType2':tempPoster.layout_style==2}">
            <img class="poster-bg" :style="{
              height:(tempPoster.layout_style==1?(contentPoster.data.second_title?(tempPoster.posterStyle.height-108)+'px':(tempPoster.posterStyle.height-60)+'px'):tempPoster.posterStyle.height),
              width:tempPoster.posterStyle.width,
              borderRadius:tempPoster.posterStyle.borderRadius,
              borderTopLeftRadius:(tempPoster.posterStyle.borderRadius?tempPoster.posterStyle.borderRadius:tempPoster.posterStyle.borderTopLeftRadius),
              borderTopRightRadius:(tempPoster.posterStyle.borderRadius?tempPoster.posterStyle.borderRadius:tempPoster.posterStyle.borderTopRightRadius)
            }" :src="contentPoster.data.bg_image"/>
            <img class="poster-content" :style="{
              height:(tempPoster.layout_style==1?(contentPoster.data.second_title?(tempPoster.posterStyle.height-108)+'px':(tempPoster.posterStyle.height-60)+'px'):tempPoster.posterStyle.height),
              width:tempPoster.posterStyle.width,
              borderRadius:tempPoster.posterStyle.borderRadius,
              borderTopLeftRadius:(tempPoster.posterStyle.borderRadius?tempPoster.posterStyle.borderRadius:tempPoster.posterStyle.borderTopLeftRadius),
              borderTopRightRadius:(tempPoster.posterStyle.borderRadius?tempPoster.posterStyle.borderRadius:tempPoster.posterStyle.borderTopRightRadius)
            }" v-if="contentPoster.data.content_image" :src="contentPoster.data.content_image"/>
            <div class="poster-title-div" :style="{
                height:(tempPoster.layout_style==0?(contentPoster.data.second_title||contentPoster.data.icon?'117px':'75px'):''),
                display:contentPoster.titleStyle.display,
                color:contentPoster.titleStyle.color,
                fontSize:contentPoster.titleStyle.fontSize,
                textAlign:contentPoster.titleStyle.textAlign}">
              <div class="poster-title-bg" :style="{
                height:(tempPoster.layout_style==0?(contentPoster.data.second_title||contentPoster.data.icon?'117px':'75px'):''),
                backgroundColor:contentPoster.titleBgStyle.backgroundColor,
                filter:contentPoster.titleBgStyle.filter,
                opacity:contentPoster.titleBgStyle.opacity,
                borderBottomLeftRadius:tempPoster.posterStyle.borderRadius,
                borderBottomRightRadius:tempPoster.posterStyle.borderRadius
              }" v-if="tempPoster.layout_style==0"></div>
              <div class="poster-title-content">
                <img class="title-img" v-if="contentPoster.data.icon && tempPoster.layout_style==0"
                     :src="contentPoster.data.icon">
                <p class="first_title" v-if="contentPoster.data.first_title"
                   :title="contentPoster.data.first_title">{{ contentPoster.data.first_title }}</p>
                <p class="second_title" v-if="contentPoster.data.second_title"
                   :title="contentPoster.data.second_title">{{ contentPoster.data.second_title }}</p>
                <span class="score" v-if="contentPoster.data.score>0 && tempPoster.layout_style==1"
                      :style="{ borderBottomRightRadius:tempPoster.posterStyle.borderRadius }">
                  {{ contentPoster.data.score.toFixed(1) }}
                </span>
              </div>
            </div>
            <img class="badge" v-if="contentPoster.data.badge_image" :src="contentPoster.data.badge_image"/>
            <p class="float-text" :title="contentPoster.data.float_text"
               v-show="!(tempPoster.layout_style==1 && contentPoster.data.score>0)"
               :style="{
                   top:(tempPoster.layout_style==1&&contentPoster.data.title_display==1?(contentPoster.data.second_title?(tempPoster.posterStyle.height-144)+'px':(tempPoster.posterStyle.height-96)+'px'):''),
                   bottom:(tempPoster.layout_style==0&&contentPoster.data.title_display==1?(contentPoster.data.second_title||contentPoster.data.icon?'117px':'75px'):''),
                   borderBottomLeftRadius:(contentPoster.data.title_display==1?tempPoster.posterStyle.borderRadius:''),
                   borderBottomRightRadius:(contentPoster.data.title_display==1?tempPoster.posterStyle.borderRadius:'') }"
               v-if="contentPoster.data.float_text && tempPoster.layout_style!=2">
              {{ contentPoster.data.float_text }}
            </p>
          </div>
        </div>
      </li>
    </ul>
  </div>
</div>
</template>

<script>
export default {
  name:'mainPage',
  data () {
    return {
      showNum:3,
      selTabId:'',
      busy: false,
      sendTemp:{},
      tabList:[],
      tempMap:{},
      contentData:[],
      smallFlag:0,
      lastComponentId:-1,
      lastWeight:-1,
      tabSmallList:[]
    }
  },
  mounted() {
    let self=this;
    self.host='http://localhost:8085/kkYiuiOperationSystem';
//    self.host='http://172.20.4.169:8074/kkYiuiOperationSystem';
//    self.host=window.localStorage.getItem('YiuiHost')
    self.checkJson=window.localStorage.getItem('checkJson')?window.localStorage.getItem('checkJson'):'[]';
    self.getTabList();
  },
  methods: {
    toSelTab(item){
      let self=this;
      console.log(item);
      self.selTabId=item.tab_id;
      self.contentData=[];
      self.lastComponentId=-1;
      self.lastWeight=-1;
      self.getTabContent();
    },
    toMoreTab(flag){
      let self=this;
      if(flag=='L'){
        if(self.smallFlag<=0){
          return;
        }
        self.smallFlag=self.smallFlag-1;
      }else if(flag=='R'){
        if(self.smallFlag>=Math.ceil(self.tabList.length/5)-1){
          return;
        }
        self.smallFlag=self.smallFlag+1;
      }
    },
    getTabList(){
      let self=this;
      let params={
        checkJson:self.checkJson
      };
      console.log(params)
      self.$http.get(self.host+'/tvPreview/baseTab',{params:params}).then((response) => {
        // 响应成功回调
        let data=response.data;
        if(data.ret.ret_code==0 || data.ret.ret_code=='0'){
//          console.log( self.sortWeight([{weight:3},{weight:1},{weight:5},{weight:7}]) );
          self.tabList=self.sortWeight(data.data);
          if(self.tabList.length>0){
            self.selTabId=self.tabList[0].tab_id;
            self.getSmallList();
            self.getTabContent();
          }
        }
      }, (response) => {
        // 响应错误回调
        console.log(response)
      });
    },
    getTabContent(){
      let self=this;
      if(self.selTabId==''){
        return;
      }
      let data={
        checkJson:self.checkJson,
        component_id: self.lastComponentId,
        last_weight: self.lastWeight,
        show_num: self.showNum,
        tab_id: self.selTabId
      };
      self.busy = true;
      self.$http.post(self.host+'/tvPreview/tabContent',data).then((response) => {
        // 响应成功回调
        let data=response.data;
        if(data.ret.ret_code==0 || data.ret.ret_code=='0'){
          let tabContentInfo=data.data;
          if(tabContentInfo.length<=0){
            self.busy = true;
          }else{
            self.busy = false;
            for(let i=0;i<tabContentInfo.length;i++){
              let temp=self.getContentObj(tabContentInfo[i]);
              if(!self.tempMap[temp.resData.template_id]){
                self.contentData.push({
                  template:{ },
                  content:temp
                });
                self.getTemplate(temp.resData);
              }else{
                self.contentData.push({
                  template:self.tempMap[temp.resData.template_id],
                  content:temp
                });
              }
              if(i==tabContentInfo.length-1){
                self.lastComponentId=temp.resData.component_id;
                self.lastWeight=temp.resData.weight;
              }
            }
          }
        }
      }, (response) => {
        // 响应错误回调
        console.log(response)
      })
    },
    getTemplate(obj){
      let self=this;
      if(self.sendTemp[obj.template_id]){
        return;
      }
      self.sendTemp[obj.template_id]=true;
      let params={
        template_id:obj.template_id
      };
      self.$http.get(self.host+'/template/queryById',{params:params}).then((response) => {
        // 响应成功回调
        let data=response.data;
        if(data.ret.ret_code==0 || data.ret.ret_code=='0'){
          self.getStyleObj(data.data,obj);
          self.checkContent(obj.template_id)
        }
      }, (response) => {
        // 响应错误回调
        console.log(response)
      });
    },
    checkContent(id){
      let self=this;
      for(let i in self.contentData){
        if(!self.contentData[i].template.data && self.contentData[i].content.resData.template_id==id){
          self.contentData[i].template=self.tempMap[id];
        }
      }
    },
    getContentObj(resData){
      let obj={
        resData:resData,
        postList:[]
      };
      for(let i=0;i<resData.posterInfoList.length;i++){
        let temp=resData.posterInfoList[i];
        let titleStyle={
          textAlign:(temp.title_location==0?'left':'center'),
          display:temp.title_display!=1?'none':''
        };
        let titleBgStyle={
          opacity:temp.title_opacity/100,
          filter:'Alpha(opacity='+temp.title_opacity+')',
          backgroundColor:temp.title_bg_color
        };
        obj.postList.push({
          data:temp,
          titleStyle:titleStyle,
          titleBgStyle:titleBgStyle
        })
      }
      return obj;
    },
    getStyleObj(data,obj){
      let self=this;
      let tempObj={};
      tempObj.data=data;
      tempObj.liStyle={
        fontFamily:data.title_font==0?'思源宋体':data.font_name,
        height:data.height+'px',
        width:(data.width-17)+'px'
      };
      tempObj.titleStyle={
        fontSize:data.title_size+'px',
        textAlign:(data.title_location==0?'left':'center'),
        display:data.title_exist_sign==0?'none':'',
        color:data.title_color
      };
      tempObj.posters=[];
      for(let i=0;i<data.posterSiteList.length;i++){
        let item=data.posterSiteList[i];
        tempObj.posters[i]={};
        tempObj.posters[i].posterData=item;
        tempObj.posters[i].layout_style=item.layout_style;
        if(tempObj.posters[i].layout_style==1){
          tempObj.posters[i].divStyle={
            height:item.height+'px',
            width:item.width+'px',
            top:item.axis_y+'px',
            left:item.axis_x+'px',
            borderBottomLeftRadius:item.radius+'px',
            borderBottomRightRadius:item.radius+'px'
          };
          tempObj.posters[i].posterStyle={
            height:item.height,
            width:item.width+'px',
            borderTopLeftRadius:item.radius+'px',
            borderTopRightRadius:item.radius+'px'
          };
        }else{
          tempObj.posters[i].divStyle={
            height:item.height+'px',
            width:item.width+'px',
            top:item.axis_y+'px',
            left:item.axis_x+'px'
          };
          tempObj.posters[i].posterStyle={
            height:item.height+'px',
            width:item.width+'px',
            borderRadius:item.radius+'px'
          };
        }
      }
      self.tempMap[obj.template_id]=tempObj;
    },
    getSmallList(){
      let self=this;
      let tabSize=5;
      if(self.tabList.length>tabSize){
        self.smallFlag=0;
        self.tabSmallList=[];
        let length=self.tabList.length;
        for(let i=0;i<Math.ceil(length/tabSize);i++){
          self.tabSmallList[i]=[];
          for(let j=0;j<tabSize;j++){
            let idx=i*tabSize+j;
            self.tabSmallList[i].push(self.tabList[idx]);
            if(idx+1==length ){
              break;
            }
          }
        }
      }
    },
    sortWeight(arr){
      if(arr.length<=0 || arr[0].weight==undefined){
        return arr;
      }
      for(let i=1;i<arr.length;i++){
        let j=i-1;
        let temp=arr[i];
        while(j>=0&&arr[j].weight<temp.weight){
          arr[j+1]=arr[j];
          j--
        }
        if(j!=i-1){
          arr[j+1]=temp;
        }
      }
      return arr;
    }
  },
  computed:{
    bgImage( ){
      let self=this;
      if(self.tabList.length>0){
        for(let i=0;i<self.tabList.length;i++){
          if(self.tabList[i].tab_id==self.selTabId){
            return self.tabList[i].bg_image?self.tabList[i].bg_image:'./static/bg.png';
          }
        }
      }
      return './static/bg.png';
    }
  }

}
</script>

<style scoped>
* {
  color: #fff;
  font-family: "思源黑体 CN Regular";
  font-weight: normal;
}

.content{
  margin-left: 96px;
}
.content-bg{
  position: fixed;
  top:0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -1;
  background-color: RGBA(0,0,0,0.7);
}
.tab-nav{
  list-style: none;
  margin: 60px 0 5px;
  overflow-x: hidden;
  white-space:nowrap;
  min-width: 1800px;
}
.tab-nav .nav-li{
  max-width: 300px;
  overflow:hidden;
  white-space:nowrap;
  text-overflow:ellipsis;
  float: left;
  cursor: pointer;
  margin-right: 60px;
  height: 66px;
  line-height: 66px;
  font-size: 45px;
  color: #ffffff;
  opacity: 0.5;
  padding: 0 25px;
  font-family: "DemiLight";
}
.tab-nav .nav-li.active{
  background-color: RGB(240,50,58);
  opacity: 1;
  border-bottom-right-radius: 15px;
  border-top-left-radius: 15px;
}
.tab-nav .nav-li:last-child{
  margin-right: 0;
}
.tab-nav .tab-more{
  width: 50px;
  height: 50px;
  font-size: 39px;
  line-height: 47px;
  border: 2px solid;
  border-radius: 50%;
  display: inline-block;
  margin-top: 6px;
  padding-left: 4px;
  cursor: pointer;
}
.tab-nav .tab-more-L{
  position: absolute;
  left: 30px;
}
.tab-nav .tab-more-R{
  margin-left: -45px;
}
.content-ul>li{
  margin-top: 60px;
  margin-bottom: 60px;
  position: relative;
  list-style: none;
}
.content-ul>li .poster-div{
  position: absolute;
}
.content-ul>li .poster-bg{
  z-index: 0;
  /*position: absolute;*/
}
.content-ul>li .poster-content{
  z-index: 1;
  position: absolute;
  top:0;
}
.content-ul>li .poster-title-div{
  position: absolute;
  bottom: 0;
}
.content-ul>li .poster-title-bg{
  position: absolute;
  z-index: 10;
}
.content-ul>li .badge{
  position: absolute;
  max-width: 100px;
  max-height: 50px;
  top: 18px;
  right: 24px;
  z-index: 10;
}
.content-ul>li .float-text{
  position: absolute;
  z-index: 10;
}
.content-ul>li .score{
  position: absolute;
  z-index: 10;
}
.content-ul>li:first-child{
  margin-top: 0;
}

.title-img{
  margin-top: 24px;
  width: 78px;
  height: 78px;
  opacity: 1;
  float: left;
  margin-right: 22px;
}
.first_title{
  overflow:hidden;
  white-space:nowrap;
  text-overflow:ellipsis;
  color: #ffffff;
  font-size: 42px;
  line-height: 42px;
  opacity: 1;
}
.second_title{
  overflow:hidden;
  white-space:nowrap;
  text-overflow:ellipsis;
  color: #ffffff;
  opacity: 0.5;
  font-size: 27px;
  line-height: 27px;
}
.float-text{
  overflow:hidden;
  white-space:nowrap;
  text-overflow:ellipsis;
  color: #e0e0e0;
  font-size: 24px;
  opacity: 1;
  background: url("../assets/bg_uplayer.png");
  background-size: 30px;
  padding-left: 18px;
  line-height: 45px;
  height: 45px;
}
/*标准风格：标题在海报内以遮罩方式展现（一级、二级标题）*/
.posterType0 .poster-title-div,.posterType0 .poster-title-bg{
  bottom: 0;
  left: 0;
  right: 0;
}
.posterType0 .poster-title-content{
  position: relative;
  z-index: 10;
  margin-left: 30px;
}
.posterType0 .poster-title-div .first_title{
  font-size: 36px;
  line-height: 75px;
  height: 75px;
}
.posterType0 .poster-title-div .second_title{
  margin-top: -5px;
  font-size: 30px;
  line-height: 30px;
}
.posterType0 .float-text{
  bottom: 0;
  left: 0;
  right: 0;
}
.posterType0 .float-text.no-title{
  bottom: 0;
}
/*分离风格：标题在海报下显示*/
.posterType1 .poster-title-div{
  width: 100%;
  box-sizing: content-box;
}
.posterType1 .poster-title-div .first_title{
  font-size: 36px;
  line-height: 36px;
  color: #a8a9a9;
  margin-top:24px;
  box-sizing: content-box;
}
.posterType1 .poster-title-div .second_title{
  font-size: 30px;
  line-height: 30px;
  color: #535455;
  margin-top:18px;
  box-sizing: content-box;
}
.posterType1 .float-text{
  left: 0;
  right: 0;
  box-sizing: content-box;
}
.posterType1 .score{
  box-sizing: content-box;
  font-size: 27px;
  color: #ffffff;
  text-align: center;
  width: 66px;
  height: 60px;
  line-height: 60px;
  top: -60px;
  right: 0;
  background-color: #F52C33;
}
/*功能风格：标题在海报内居中显示*/
.posterType2 .poster-title-div{
  bottom: 0;
  left: 0;
  right: 0;
  top: 0;
  display: table;
  height: 100%;
  width: 100%;
}
.posterType2 .poster-title-content{
  display: table-cell;
  vertical-align: middle;
  text-align: center !important;
}
.posterType2 .poster-title-content .first_title{
  display: block;
  text-align: center !important;
}
.posterType2 .poster-title-content .second_title{
  display: block;
  opacity: 1;
  margin-top: 15px;
  text-align: center !important;
}
</style>
