<template>
  <div>
    <Login style="position: absolute;z-index: 2" v-if="login.showLogin" v-bind:close_frame="login.frame" v-on:closeFrame="askLoginOrRegister($event)"></Login>
    <Register style="position: absolute;z-index: 2" v-if="login.showRegister" v-bind:close_frame="login.frame" v-on:closeFrame="askLoginOrRegister($event)"></Register>
    <Forget style="position: absolute;z-index: 2" v-if="login.showForget" v-bind:close_frame="login.frame" v-on:closeFrame="askLoginOrRegister($event)"></Forget>
    <div style="background-color: #FFFFFF;padding: 10px" v-bind:style="{filter:'blur('+login.blur_num+'px)'}">
      <Menu ref="menu" v-bind:frame="login.frame" v-on:showFrame="askLoginOrRegister($event)"></Menu>
      <div id="article area" style="width: 82%;margin-left: auto;margin-right: auto;">
        <Breadcrumb separator=">" style="margin-bottom: 15px">
          <BreadcrumbItem to="/">全部</BreadcrumbItem>
          <BreadcrumbItem to="/">{{first_tag}}</BreadcrumbItem>
          <BreadcrumbItem>{{second_tag}}</BreadcrumbItem>
        </Breadcrumb>
        <p style="font-weight: bold;font-size: 32px;color: #464c5b;margin-bottom: 15px">
          <Row>
            <Col span="21">{{title}}</Col>
            <Col span="3" v-bind:style="star_style">
              <Tooltip v-bind:content="this.tooltip" style="position: relative;left: 20px;">
                <Icon v-bind:style="this.style" type="md-heart" @click="starOrCancel()"/>
              </Tooltip>
              <span style="font-size: 15px;position: relative;left: 20px;">{{this.status}}</span>
            </Col>
          </Row>
        </p>
        <div class="markdown-body parent" id="content" style="font-size: 16px;background-color: rgb(245,246,247);padding: 20px 20px 20px 20px;margin-bottom: 15px;min-height: 50px" v-html="content" v-loading="isShow" loading-text="正在加载...">
        </div>
        <Card style="margin-bottom: 15px">
          <Row>
            <Col span="3">
            <Row style="text-align: center">
              <img style="width:40px;cursor: pointer;border-radius: 50%" v-bind:data-id="this.author_id" v-bind:src="this.author_photo" @click="jumpToPersonal(author_id)">
            </Row>
            <Row style="text-align: center">
              <span>本文作者</span>
            </Row>
            </Col>
            <Col span="3">
            <Row style="margin-top:16px;text-align: center">
              <span> &nbsp;{{this.author_name}}</span>
            </Row>
            <Row style="text-align: center">
              <Time v-bind:time="this.publish_time" :interval="1"/>
            </Row>
            </Col>
            <Col span="3">
            <Row style="margin-top:16px;text-align: center">
              <span> <Icon type="md-star" />&nbsp;{{this.starNum}}</span>
            </Row>
            <Row style="text-align: center">
              <span>关注帖子数</span>
            </Row>
            </Col>
            <Col span="3">
            <Row style="margin-top:16px;text-align: center">
              <span><Icon type="md-clipboard" />&nbsp;{{this.commentNum}}</span>
            </Row>
            <Row style="text-align: center">
              <span>回复帖子数</span>
            </Row>
            </Col>
            <Col span="3">
            <Row style="margin-top:16px;text-align: center">
              <span> <Icon type="md-people" />&nbsp;{{this.accessNum}}</span>
            </Row>
            <Row style="text-align: center">
              <span>被访问次数</span>
            </Row>
            </Col>
            <Col span="3">
            <Row style="margin-top:16px;text-align: center">
              <span> <Icon type="md-document" />&nbsp;{{this.browseNum}}</span>
            </Row>
            <Row style="text-align: center">
              <span>浏览帖子数</span>
            </Row>
            </Col>
            <Col span="3">
            <Row style="margin-top:16px;text-align: center">
              <span> <Icon type="md-cloud-upload" />&nbsp;{{this.publishNum}}</span>
            </Row>
            <Row style="text-align: center">
              <span>发布帖子数</span>
            </Row>
            </Col>
            <Col span="3">
            <Row style="margin-top:16px;text-align: center">
              <span> <Icon type="md-heart" />&nbsp;{{this.collectNum}}</span>
            </Row>
            <Row style="text-align: center">
              <span>收藏帖子数</span>
            </Row>
            </Col>
          </Row>
        </Card>
        <div v-bind:style="comment_style">
          <p style="margin-bottom: 15px;color: #4285f4;padding-left: 10px;font-size: 16px">添加回帖内容...</p>
          <div style="margin-bottom: 15px">
            <Row>
              <Col span="2"><img style="width:50px;padding-left: 10px;cursor: pointer;border-radius: 50%" v-bind:src="this.user_photo" v-bind:data-id="this.user_id" @click="jumpToPersonal(user_id)"></Col>
              <Col span="22">
              <div id="editor">
                <editorbar v-model="editor.info" :isClear="isClear"></editorbar>
              </div>
              </Col>
            </Row>
          </div>
          <div style="margin-bottom: 30px">
            <Button type="primary" @click="addComment()" style="position: relative;left: 92%;">提交评论</Button>
          </div>
        </div>
        <div v-bind:style="tooltip_style">
          <p style="margin-bottom: 15px;color: #4285f4;padding-left: 10px;font-size: 16px">登录之后可以添加回复...</p>
        </div>
        <table id="comments" style="width: 100%">
          <tr style="margin-bottom: 15px">
            <span style="font-size: 16px;font-weight: bold">{{total_comments.length+"回帖"}}</span>
          </tr>
          <Divider style="margin-bottom: 15px"/>
          <tr v-for="(comment,index) in show_comments" class="page">
            <Card style="margin-bottom:15px">
              <Row>
                <Col span="2">
                <img style="width:40px;cursor: pointer;border-radius: 50%" v-bind:data-id="comment.user_id" v-bind:src="comment.user_photo" @click="jumpToPersonal(comment.user_id)">
                </Col>
                <Col span="20">
                <Row>
                  <p>
                    <span>{{comment.user_name}}</span>&nbsp;
                    <Time v-bind:time="comment.time" :interval="60"/>
                  </p>
                </Row>
                <Row>
                  <div v-html="comment.content"></div>
                </Row>
                </Col>
              </Row>
            </Card>
          </tr>
          <tr>
            <td colspan="4" style="background-color: #f8f8f9;padding: 7px">
              <Page @on-change="changePage" :current="current_page" v-bind:total="total_comments.length" :page-size="page_size" show-elevator />
            </td>
          </tr>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
  import Menu from '../components/Menu/Menu'
  import Editorbar from "../components/wangeditor/wangeditor";
  import 'mavon-editor/dist/markdown/github-markdown.min.css'
  import Login from '../components/user/LoginPage'
  import Register from '../components/user/RegisterPage'
  import Forget from '../components/user/ForgetPage'
  export default {
    name: "DetailPage",
    components:{
      Editorbar,
      Menu,Login,Register,Forget,
    },
    async mounted(){
      this.post_id=sessionStorage.getItem("post_detail_id");
      if(sessionStorage.getItem("username")!=null&&sessionStorage.getItem("username")!=""){
        this.user_id=sessionStorage.getItem("username");
        await this.$axios.post('/server/C_User/judgeCollect', {
          currentUser: this.user_id,
          param: this.post_id
        }).then(re => {
          if (re.data) {
            this.isStar = true;
            this.tooltip = '取消收藏';
            this.status = '已收藏';
            this.style = 'cursor:pointer;color:darkred';
          } else {
            this.isStar = false;
            this.tooltip = '收藏该帖';
            this.status = '未收藏';
            this.style = 'cursor:pointer;color:rgb(70, 76, 91)';
          }
        });
        await this.$axios.post('/server/getUserInfo',{email:this.user_id}).then(re=>{
          let data=re.data;
          this.user_name=data.nickname;
          this.user_photo=data.imageUrl;
        }).catch((err)=>{
          console.log("读取用户信息失败");
        });
        this.comment_style="display:block";
        this.star_style="display:block";
        this.tooltip_style="display:none";
      }else {
        this.comment_style="display:none";
        this.star_style="display:none";
        this.tooltip_style="display:block";
      }
      await this.$axios.post('/server/post/readArticle',{postID:this.post_id}).then(re=>{
        let data=re.data;
        this.title=data.postTitle;
        this.author_id=data.author;
        this.first_tag=data.postCategory;
        this.second_tag=data.postTag;
        this.content=data.content;
        this.publish_time=data.publish_time;
        let comments=data.remark_content;
        for(let i=0;i<comments.length;i++) {
          this.total_comments.push({
            user_id:comments[i].reviewer,
            user_photo: comments[i].review_img,
            user_name: comments[i].nickname,
            time: comments[i].remark_time,
            content: comments[i].remark_content
          });
        }
      }).catch((err)=>{
        console.log("读取文章内容失败");
      });
      await this.$axios.post('/server/getUserInfo',{email:this.author_id}).then(re=>{
        let data=re.data;
        this.author_name=data.nickname;
        this.author_photo=data.imageUrl;
        this.starNum=data.interestUserNums;
        this.commentNum=Math.round(Math.random()*50)+50;
        this.accessNum=Math.round(Math.random()*50)+50;
        this.browseNum=Math.round(Math.random()*50)+50;
        this.publishNum=data.releasedNum;
        this.collectNum=data.collectPostNums;
      }).catch((err)=>{
        console.log("读取用户信息失败");
      });
      this.isShow=false;
      if(this.total_comments.length<this.page_size){
        this.show_comments=this.total_comments;
      }else{
        this.show_comments=this.total_comments.slice(0,this.page_size);
      }
    },
    data(){
      return {
        author_id:'',
        author_photo:'',
        author_name:'',
        user_id:'',
        user_name:'',
        user_photo:'',
        post_id:'',
        starNum:0,
        commentNum:0,
        accessNum:0,
        browseNum:0,
        publishNum:0,
        collectNum:0,
        title:'',
        content:'',
        first_tag:'',
        second_tag:'',
        publish_time:0,
        isStar:true,
        tooltip:'收藏该帖',
        status:'未收藏',
        style:'cursor:pointer;color:rgb(70, 76, 91)',
        comment_style:'display:none',
        star_style:'display:none',
        tooltip_style:'display:block',
        isShow:true,
        login:{
          showLogin:false,
          showRegister:false,
          showForget:false,
          blur_num:0,
          frame:''
        },
        editor: {
          info:'想对作者说些什么...'
        },
        isClear:false,
        page_size:15,
        current_page:1,
        show_comments:[],
        total_comments:[]
      }
    },
    methods:{
      //在特定的标签下切换页面
      changePage(page){
        let start=(page-1)*this.page_size;
        let end=page*this.page_size;
        this.show_comments=this.total_comments.slice(start,end);
      },
      async getData(){
        if(sessionStorage.getItem("username")!=null){
          this.user_id=sessionStorage.getItem("username");
          await this.$axios.post('/server/C_User/judgeCollect', {
            currentUser: this.user_id,
            param: this.post_id
          }).then(re => {
            if (re.data) {
              this.isStar = true;
              this.tooltip = '取消收藏';
              this.status = '已收藏';
              this.style = 'cursor:pointer;color:darkred';
            } else {
              this.isStar = false;
              this.tooltip = '收藏该帖';
              this.status = '未收藏';
              this.style = 'cursor:pointer;color:rgb(70, 76, 91)';
            }
          });
          await this.$axios.post('/server/getUserInfo',{email:this.user_id}).then(re=>{
            let data=re.data;
            this.user_name=data.nickname;
            this.user_photo=data.imageUrl;
          }).catch((err)=>{
            console.log("读取用户信息失败");
          });
          this.comment_style="display:block";
          this.star_style="display:block";
          this.tooltip_style="display:none";
        }else {
          this.comment_style="display:none";
          this.star_style="display:none";
          this.tooltip_style="display:block";
        }
        this.post_id=sessionStorage.getItem("post_detail_id");
        await this.$axios.post('/server/post/readArticle',{postID:this.post_id}).then(re=>{
          let data=re.data;
          this.title=data.postTitle;
          this.author_id=data.author;
          this.first_tag=data.postCategory;
          this.second_tag=data.postTag;
          this.content=data.content;
          this.publish_time=data.publish_time;
          let comments=data.remark_content;
          for(let i=0;i<comments.length;i++) {
            this.total_comments.push({
              user_id:comments[i].reviewer,
              user_photo: comments[i].review_img,
              user_name: comments[i].nickname,
              time: comments[i].remark_time,
              content: comments[i].remark_content
            });
          }
        }).catch((err)=>{
          console.log("读取文章内容失败");
        });
        if(sessionStorage.getItem("username")!=null) {

        }else{

        }
        await this.$axios.post('/server/getUserInfo',{email:this.author_id}).then(re=>{
          let data=re.data;
          this.author_name=data.nickname;
          this.author_photo=data.imageUrl;
          this.starNum=data.interestUserNums;
          this.commentNum=Math.round(Math.random()*50)+50;
          this.accessNum=Math.round(Math.random()*50)+50;
          this.browseNum=Math.round(Math.random()*50)+50;
          this.publishNum=data.releasedNum;
          this.collectNum=data.collectPostNums;
        }).catch((err)=>{
          console.log("读取用户信息失败");
        });
        this.isShow=false;
        if(this.total_comments.length<this.page_size){
          this.show_comments=this.total_comments;
        }else{
          this.show_comments=this.total_comments.slice(0,this.page_size);
        }
      },
      addComment(){
        if(sessionStorage.getItem("username")==null){
          window.scrollTo(0, 0);
          document.documentElement.scrollTop = document.body.scrollTop = 0;
          this.$el.parentNode.scrollTop = 0;
          this.askLoginOrRegister('login');
          return
        }
        let comment=$(".w-e-text").html();
        $(".w-e-text").html("");
        let time=(new Date()).getTime();
        let data={
          user_photo:this.user_photo,
          user_name:this.user_name,
          time:time,
          content:comment
        };
        console.log(this.user_name);
        this.total_comments.unshift(data);
        this.changePage(1);
        this.$axios.post('/server/post/remark',{post_id:this.post_id,reviewer:this.user_id,remark_content:comment}).catch((err)=>{
          console.log("发表评论出错");
        })
      },
      starOrCancel(){
        if(this.isStar){
          this.isStar=false;
          this.tooltip='收藏该帖';
          this.status='未收藏';
          this.style='cursor:pointer;color:rgb(70, 76, 91)';
          this.$axios.post('/server/C_User/cancelCollectPost',{currentUser:this.user_id,param:this.post_id}).catch((err)=>{
            console.log("收藏帖子错误");
          });
        }else{
          this.isStar=true;
          this.tooltip='取消收藏';
          this.status='已收藏';
          this.style='cursor:pointer;color:darkred';
          this.$axios.post('/server/C_User/collectPost',{currentUser:this.user_id,param:this.post_id}).catch((err)=>{
            console.log("收藏帖子错误");
          });
        }
      },
      jumpToPersonal(user_id){
        // console.log(user_id);
        // var user_id=$(this).attr("src");
        // console.log(user_id);
        sessionStorage.setItem("author_email",user_id);
        window.location.href='/author';
      },
      //判断该打开login窗口还是register窗口,或者关闭该窗口
      askLoginOrRegister(frame){
        if(frame=='login'){
          this.login.showLogin=true
          this.login.showRegister=false
          this.login.showForget=false
          this.login.blur_num=10
        }
        else if(frame=='register'){
          this.login.showLogin=false
          this.login.showRegister=true
          this.login.showForget=false
          this.login.blur_num=10
        }
        else if(frame=='forget'){
          this.login.showLogin=false
          this.login.showRegister=false
          this.login.showForget=true
          this.login.blur_num=10
        }
        else if(frame=='close'){
          this.login.showLogin=false
          this.login.showRegister=false
          this.login.showForget=false
          this.login.blur_num=0
          if(sessionStorage.getItem("username")!=null && sessionStorage.getItem("username")!=''){
            this.$router.go(0)
          }
        }
        this.$refs.menu.getUser()
      }
    },
    created(){
      //this.getData();
    }
  }
</script>

<style scoped>
  .w-e-text-container{
    height: 700px !important;
  }
  Row{ text-align: center}
</style>
