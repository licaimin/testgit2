<template>
  <div class="home">
    <div class="content">
      <div class="information">
        <div class="userPicture">
          <img :src="baseInfo.avatar" alt="用户头像">
        </div>
        <span>{{baseInfo.nickname}}</span>
        <p>{{baseInfo.personalDescription}}</p>
      </div>
      <el-row class="atten">
        <el-col :span="4" :offset="6">
          <span>{{baseInfo.popularity}}</span>
          <p>人气</p>
        </el-col>
        <el-col :span="4">
          <router-link :to="{path:'/personalHome/focusfans',name:'focusfans',params:{id: userId}}" class="personalTag">
            <span>{{baseInfo.fansCount}}</span>
            <p>粉丝</p>
          </router-link>
        </el-col>
        <el-col :span="4">
          <router-link :to="{path:'/personalHome/focusfans',name:'focusfans',params:{id: userId}}" class="personalTag">
            <span>{{baseInfo.followCount}}</span>
            <p>关注</p>
          </router-link>
        </el-col>
      </el-row>
      <div class="tabBox">
        <el-row class="tab-top">
          <el-col :span="3" :offset="9" class="tab-tag active"><p><a href="javascript:;" @click="changeTab(0)">关注的人</a></p><span></span></el-col>
          <el-col :span="3" class="tab-tag"><p><a href="javascript:;" @click="changeTab(1)">粉丝</a></p><span></span></el-col>
        </el-row>
        <div class="tab-content" id="pills-tabContent" style="background: #f6f6f6;position: relative;top: -0.5rem;">
          <!--关注-->
          <div class="tab-show active" style="padding-top: 32px;padding-bottom: 10px;">
            <b-card-group deck class="focusList">
              <div v-for="item in focusList">
                <b-card style="max-width: 20rem;"
                        class="mb-2 focus">
                  <b-img :src="item.avatar" @click="checkPeople(item.id)"
                         style="cursor: pointer"
                         rounded="circle" width="64" height="64"
                         alt="img" class="m-1 focusHeadPicture" />
                  <p class="focusName" style="cursor: pointer" @click="checkPeople(item.id)">{{item.nickname}}</p>
                  <p class="focusSlogan">{{item.personalDescription}}</p>
                  <b-button variant="primary" class="followTrue"
                            v-if="!item.islike" @click="followThem(true, item)">
                    <span>关注</span>
                  </b-button>
                  <b-button variant="primary" class="followFalse"
                            v-if="item.islike" @click="followThem(false, item)">
                    <span>已关注</span>
                  </b-button>
                </b-card>
              </div>
            </b-card-group>
            <b-pagination v-if="focusList != undefined && focusList.length!=0"
                              :total-rows="focusTotal" :per-page="20"
                              v-model="focusPageNum" @click="getFans"
                              align="center" hide-goto-end-buttons  />
          </div>

          <!--粉丝-->
          <div class="tab-show" style="padding-top: 32px;padding-bottom: 10px;">
            <b-card-group deck class="fansList">
              <div v-for="item in fansList">
                <b-card style="max-width: 20rem;"
                        class="mb-2 fans">
                  <b-img :src="item.avatar" @click="checkPeople(item.id)"
                         style="cursor: pointer"
                         rounded="circle" width="64" height="64"
                         alt="img" class="m-1 fansHeadPicture" />
                  <p class="nickname" style="cursor: pointer" @click="checkPeople(item.id)">{{item.nickname}}</p>
                  <p class="fansSlogan">{{item.personalDescription}}</p>
                  <b-button variant="primary" class="followTrue"
                            v-if="!item.islike" @click="followThem(true, item)">
                    <span>关注</span>
                  </b-button>
                  <b-button variant="primary" class="followFalse"
                            v-if="item.islike" @click="followThem(false, item)">
                    <span>已关注</span>
                  </b-button>
                </b-card>
              </div>
            </b-card-group>
            <b-pagination :total-rows="fansTotal" :per-page="20"
                          v-model="fansPageNum" @click="getFollow"
                          v-if="fansList != undefined && fansList.length!=0"
                          align="center" hide-goto-end-buttons  />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  let focusFanTabTag,focusFanTabShow;
  if (typeof window.onload != 'function'){
    window.onload = function () {
      focusFanTabTag = document.getElementsByClassName('tab-tag');
      focusFanTabShow = document.getElementsByClassName('tab-show');
    };
  }else {
    let oldLoad = window.onload;
    window.onload = function () {
      oldLoad();
      focusFanTabTag = document.getElementsByClassName('tab-tag');
      focusFanTabShow = document.getElementsByClassName('tab-show');
    };
  }

  export default {
    name: "focusfans",
    data: function () {
      return {
        userId: '',
        currentTab: 0,
        baseInfo:{id: '', nickname: '',avatar:'', personalDescription:'',fansCount: '',followCount: '',popularity:''},
        currentPage: 1,
        fansTotal: 20,
        fansPageNum:1,
        fansList:[],
        focusTotal: 20,
        focusPageNum:1,
        focusList:[]
      }
    },
    methods:{
      // 更换标签
      changeTab:function(tabIndex){
        if (tabIndex == this.currentTab) return true;
        // 当前的标签
        focusFanTabTag[tabIndex].className += ' active';
        focusFanTabShow[tabIndex].className += ' active';
        // 上个标签
        let lastTabClass = focusFanTabTag[this.currentTab].className;
        let lastTabShowClass = focusFanTabShow[this.currentTab].className;
        focusFanTabTag[this.currentTab].className = lastTabClass.replace(/ active/,'');
        focusFanTabShow[this.currentTab].className = lastTabShowClass.replace(/ active/,'');
        this.currentTab = tabIndex;
      },
      // 访问他人资料
      checkPeople:function(id){
        this.$router.push({path: '/personalHome',name: 'personalHome', params:{id: id}})
      },
      //获得基本信息
      getBaseInfo:function(){
        let self = this;
        this.$axios({
          method: 'get',
          baseURL: '/api',
          url: '/profile/baseInfo/' + self.userId
        }).then(function (res) {
          self.baseInfo = res.data.data;
        }).catch(function (err) {
          self.$message('获取基本信息失败');
        })
      },
      // 粉丝初始化
      getFans:function(){
        let self = this;
        this.$axios({
          method: 'get',
          baseURL: '/api/profile/fans',
          params:{
            pageNum: self.fansPageNum,
            pageSize: 20,
            userId: self.userId,
          }
        }).then((res)=>{
          self.fansTotal = res.data.data.total;
          self.fansList = res.data.data.rows;
        }).catch((err)=>{
          self.$message('获取数据失败');
        })
      },
      // 关注初始化
      getFollow:function(){
        let self = this;
        this.$axios({
          method: 'get',
          baseURL: '/api/profile/follows',
          params:{
            pageNum: self.focusPageNum,
            pageSize: 20,
            userId: self.userId,
          }
        }).then((res)=>{
          self.focusTotal = res.data.data.total;
          self.focusList = res.data.data.rows;
        }).catch((err)=>{
          self.$message('获取数据失败');
        })
      },
      follow:function(item){
        let self = this;
        let id = item.id
        this.$axios({
          method: 'post',
          baseURL: '/api/follow',
          url: '/'+ id
        }).then((res)=>{
          console.log(res);
          item.islike = res.data.data.islike;
        }).catch((err)=>{
          self.$message('网络错误');
        })
      },
      cancelFollow:function(item){
        let self = this;
        let id = item.id;
        this.$axios({
          method: 'del',
          baseURL: '/api/follow',
          url: '/'+ id,
        }).then((res)=>{
          item.islike = 0;
        }).catch((err)=>{
          self.$message('网络错误');
        })
      },
      followThem: function (state, item) {
        if (state === true){
          this.follow(item);
          // item.islike = !item.islike;
        }else if (state === false){
          this.cancelFollow(item);
          // item.islike = !item.islike;
        }
      }
    },
    created:function () {
      let self = this;
      this.userId = this.$route.params.id;
      this.getBaseInfo();
      this.getFollow();
      this.getFans();
    },
    deactivated() {
      this.$destroy();
    }
  }
</script>

<style scoped>
  .home{
      padding: 0;
      background-color: #f5f5f5;
  }
  .content{
    overflow: hidden;
    zoom: 1;
    font-size: 16px;
    margin: 0 auto;
    background-color: #fff;
  }
  .information{
    margin-top: 64px;
    text-align: center;
  }
  .userPicture{
    overflow: hidden;
    margin: auto auto;
    width: 120px;
    height: 120px;
    text-align: center;
    border-radius: 50%;
  }
  .userPicture img{
    width: 100%;
    height: 100%;
  }
  .information span{
    display: inline-block;
    font-size: 1.5rem;
    line-height: 24px;
    margin-top: 24px;
    margin-bottom: 16px;
  }
  .information p{
    font-size: 16px;
  }
  .atten{
    margin-top: 64px;
    text-align: center;
  }
  .personalTag{
    text-decoration: none;
    color: #515151;
  }
  .personalTag:hover{
    color: blue;
  }
  .tabBox{
    margin-top: 48px;
  }
  .tab-top{
    background: #fff;
    height: 52px;
  }
  .tab-tag a{
    padding: 10px;
    text-decoration: none;
    font-size: 16px;
    color:#999999;
    font-weight: bold;
  }
  .tab-tag.active a{
    color: #515151;
  }
  .tab-tag span{
    margin: 0 auto;
    margin-top: 5px;
    display:block;
    height: 4px;
    width:32px;
    background: inherit;
  }
  .tab-tag.active span{
    background: #4959F6;
  }
  .tab-show{
    width: 100%;
    max-width: 1216px;
    margin: 0 auto;
    display: none;
  }
  .tab-show.active{
    display: block;
  }
  .tabBox .nav .active span{
    background-color: #4959F6;
  }
  .category .active{
    color: #fff;
  }
  .work div{
    padding: 0;
  }
  .work p{
    margin-top: 16px;
    font-size: 14px;
    line-height: 16px;
    color: #515151;
  }
  .author img{
    width: 24px;
    height: 24px;
    border-radius: 50%;
  }
  .author strong{
    font-size: 12px;
    line-height: 12px;
    font-weight: normal;
    margin-left: 16px;
  }
  .author span{
    position: absolute;
    right: 11px;
    bottom: 18px;
    line-height: 12px;
    font-size: 12px;
    color: #C1C1C1;
  }
  .fansList, .focusList{
    text-align: center;
  }
  .fans, .focus{
    width: 208px;
    height: 208px;
  }
  .fans .fansName, .focus .focusName{
    margin-top: 12px;
    margin-bottom: 8px;
    font-size: 16px;
    line-height: 16px;
    color: #515151;
    font-weight: bold;
  }
  .fans .fansSlogan, .focus .focusSlogan{
    font-size: 14px;
    color: #999999;
    line-height: 14px;
  }
  .followTrue, .followFalse{
    padding: 0;
    border: none;
    height: 32px;
    width: 64px;
    font-size: 14px;
    line-height: 32px;
  }
  .followFalse{
    background-color: #F6F6F6;
    color: #C1C1C1;
  }
</style>
