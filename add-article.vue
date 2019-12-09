<template>
    <div class="slide-in " :class="{ 'show' : show, 'hide' : !show }">
        <Header :style="{padding:'0px',background:'#fff',position:'fixed',zIndex:'999'}">
            <p v-if="!articleData.Id" class="title">新增文章</p>
            <p v-else class="title">修改文章</p>
            <span class="close" :class="{ 'closeShow' : show , 'closeHide' : !show}" @click="close">
                <Icon type="ios-close" size="76" />
            </span>
        </Header>
        
        <Content class="article-content">
             <Row class="my-row">
                    <Col span="24">
                        <Form :model="articleData" :rules="ruleValidate" :label-width="150">
                            <div class="article-attribute">
                                <div class="extend-span"></div>
                                <p class="extend-nature">基本信息</p>
                            </div>
                            <FormItem label="选择栏目" prop="Column" v-if="show">
                                <Cascader change-on-select filterable
                                    v-model="columnPath"
                                    :style="{width:'100%',textAlign:'left'}"
                                    :data="selectList" 
                                    @on-change="handleChange">
                                </Cascader>
                            </FormItem>                

                            <FormItem label="添加图片" prop="Cover">
                                <div class="select-format" >
                                    <Upload 
                                        v-if="!coverPhoto"
                                        :action="`${baseUrl}api/app/img/upload`" 
                                        :headers="headers"
                                        :show-upload-list="false"
                                        :format="['jpg','jpeg','png']"
                                        :on-success="handleSuccess"
                                        :before-upload="handleUpload"
                                        :on-error="handleError"
                                        :on-exceeded-size="handleMaxSize" >
                                        <div class="format-item" style="border:1px solid #dcdee2;text-align:center;padding:8px 14px;cursor:pointer;" >
                                            <Icon type="md-add" color="#dcdee2" size="28"/>
                                            <p style="color:#dcdee2;font-size:12px;">添加图片</p>
                                        </div>
                                    </Upload>
                                    <div class="format-item format-img" style="position:relative;text-align:left;" v-if="coverPhoto">
                                        <Icon class="delete-cover" type="md-close-circle" @click="deleteCoverPhoto"/>
                                        <img :src="articleData.Cover" :style="{width:'78px',height:'82px'}" />
                                    </div>
                                    <p style="text-align:left;">建议比例3：2，大小请勿超过2M</p>
                                </div>
                            </FormItem>

                            <FormItem label="文章标题" prop="Title">
                                <Input v-model="articleData.Title" placeholder="标题 (标题不超过46个字)" />
                            </FormItem>

                            <FormItem label="文章正文" prop="Content" class="article-body">
                                <div class="my-quill-editor">
                                    <Upload
                                        :action="`${baseUrl}api/app/img/upload`" 
                                        :headers="headers" 
                                        :show-upload-list="false"
                                        :style="{display:'none'}"
                                        :on-success="handleQuillSuccess"
                                        :format="['jpg','jpeg','png','gif']"
                                        :max-size="2048"
                                        class="quill-upload"
                                        multiple>
                                        <Button icon="ios-cloud-upload-outline" ></Button>
                                    </Upload>
                                    <quill-editor 
                                        v-model="articleData.Content"
                                        ref="myQuillEditor"
                                        :options="editorOption" >
                                    </quill-editor>
                                </div>
                            </FormItem>

                            <FormItem label="是否发布" prop="Status" :style="{textAlign:'left'}">
                                <i-switch size="large" v-model="articleData.Status == '1' ? true : false" @on-change="changeStatus">
                                    <span slot="open">ON</span>
                                    <span slot="close">OFF</span>
                                </i-switch>
                            </FormItem>

                            <div class="article-attribute" v-if="moduleData.length != 0">
                                <div class="extend-span"></div>
                                <p class="extend-nature">栏目扩展属性</p>
                            </div>

                            <FormItem prop="objData"  v-for="(item,index) in moduleData" :key="index">
                                <span slot="label"><i class="must-symbol" :class="item.IsNecessary ? '' : 'noNecessary'">*</i>{{item.DisplayName}}</span>
                                <div class="select-format">
                                    <Input v-model="articleData.objData[item.Id]" placeholder="(输入内容不超过46个字)" />
                                    <p v-if="objData_error_info" style="color:#ed4014;">扩展数据对象不能为空</p>
                                </div>
                            </FormItem>
                            
                            <div class="article-attribute">
                                <div class="extend-span"></div>
                                <p class="extend-nature">扩展属性</p>
                            </div>
                            <FormItem label="子标题">
                                <Input v-model="articleData.SubTitle" placeholder="子标题 (子标题不超过46个字)"/>
                            </FormItem>

                            <FormItem label="关键字">
                                <Input v-model="articleData.Keywords" placeholder="关键字 (关键字不超过16个字)"/>
                            </FormItem>

                            <FormItem label="标签">
                                <Input v-model="articleData.Tag" placeholder="标签 (标签不超过12个字)"/>
                            </FormItem>

                            <FormItem label="摘要">
                                <Input v-model="articleData.Summary" type="textarea" placeholder="摘要 (摘要不超过200个字)"/>
                            </FormItem>

                            <FormItem label="作者">
                                <Input v-model="articleData.Author" placeholder="作者"/>
                            </FormItem>

                            <FormItem label="来源">
                                <Input v-model="articleData.Source" placeholder="文章来源"/>
                            </FormItem>

                            <FormItem label="跳转链接">
                                <Input v-model="articleData.Redirect" placeholder="跳转链接"/>
                            </FormItem>

                            <FormItem label="发布时间" prop="Time">
                                <DatePicker type="datetime" placeholder="选择发布时间" :value="articleData.PublishTime" @on-change="handlechangeDate" :style="{display:'block',width:'200px'}"></DatePicker>
                            </FormItem>

                            <FormItem label="排序"  prop="Sort" :style="{textAlign:'left'}">
                                <Input v-model="articleData.Sort" type="number" :style="{width:'200px'}"/>
                            </FormItem>
                            
                        </Form>
                    </Col>
             </Row>
            
        </Content>

        <Footer :style="{padding:'0px'}">
            <div class="info-footer">
                <button class="ant-btn ant-btn-primary" @click="submit">保存</button>
                <button class="ant-btn ant-btn-default" @click="close">取消</button>
            </div>
        </Footer>
        
    </div>
    
</template>
<script>
import { getToken } from '@/libs/util'
import config from '@/config'
const { baseUrl } = config
//调用vue-quill-editor编辑器
import { quillEditor } from "vue-quill-editor";
import { addQuillTitle } from '@/libs/quill-title'
import 'quill/dist/quill.core.css';
import 'quill/dist/quill.snow.css';
import 'quill/dist/quill.bubble.css';

import { saveArticle,getArticleInfo,getColumnObject } from '@/api/data'
import { mapGetters } from 'vuex'
import { getPathByKey } from '@/libs/util'
export default {
    props:{
        show:Boolean,
        columnList:Array
    },
    data () {
        return{
            baseUrl: baseUrl.dev,
            headers:{
                    Authorization: !!getToken() ? ('Bearer ' + getToken()) : ''
            },
            editorOption: {     //quill编辑器功能选择
                modules:{
                    toolbar:{
                        container:[
                            ['bold', 'italic', 'underline', 'strike'],    //加粗，斜体，下划线，删除线
                            ['blockquote', 'code-block'],     //引用，代码块
                            [{ 'header': 1 }, { 'header': 2 }],        // 标题，键值对的形式；1、2表示字体大小
                            [{ 'list': 'ordered'}, { 'list': 'bullet' }],     //列表
                            [{ 'script': 'sub'}, { 'script': 'super' }],   // 上下标
                            [{ 'indent': '-1'}, { 'indent': '+1' }],     // 缩进
                            [{ 'direction': 'rtl' }],             // 文本方向
                            [{ 'size': ['small', false, 'large', 'huge'] }], // 字体大小
                            [{ 'header': [1, 2, 3, 4, 5, 6, false] }],     //几级标题
                            [{ 'color': [] }, { 'background': [] }],     // 字体颜色，字体背景颜色
                            [{ 'font': [] }],     //字体
                            [{ 'align': [] }],    //对齐方式
                            ['clean'],    //清除字体样式
                            ['image','video']    //上传图片、上传视频
                        ],
                        handlers: {
                            'image': function (value) {
                                if (value) {
                                    // 调用iview图片上传
                                    document.querySelector('.quill-upload .ivu-btn').click()
                                } else {
                                    this.quill.format('image', false);
                                }
                            }
                        }
                    }
                    
                },
                placeholder:'请在这里输入'
            }, 
            coverPhoto:false,   
            moduleData:[],      //分类id绑定的对象数据信息
            articleData:{
                Id:'',
                Title:'',
                SubTitle:'',
                Keywords:'',
                Tag:'',
                Summary:'',
                Author:'',
                Source:'',
                Redirect:'',
                PublishTime:'',
                Sort:0,
                CategoryId:'',
                Status:'1',
                Content:'',
                Cover:'',
                objData:{},
                CategoryName:''        //仅在修改时有值
            },
            ruleValidate:{
                Title:[
                    { required: true, message: '文章标题不能为空', trigger: 'blur' }
                ],
                Content:[
                    { required: true, message: '文章内容不能为空', trigger: 'blur' }
                ],
                Time:[
                    { required: true, message: '发布时间不能为空', trigger: 'blur' }
                ],
                Sort:[
                    { required: true, message: '排序不能为空', trigger: 'blur' }
                ]

            },
            columnPath:[],  //级联选择器路径
            objData_error_info:false    //栏目扩展对象错误提示
        }
    },
    watch:{
        show (val){
            this.articleData.PublishTime = new Date().toLocaleString('chinese',{hour12:false})
        }
    },
    components :{
        quillEditor,
    },
    computed: {
        ...mapGetters([
            'appId'
        ]),
        editor() {
            return this.$refs.myQuillEditor.quill;
        },
        selectList (){
            return this.convertTree(this.columnList)
        }
    },
    methods: {
        changeStatus (data){
            data ? data = '1' : data = '0'
            this.articleData.Status = data
        },
        // 准备编辑器
        onEditorReady(editor) { 
        },
        //添加上传图片限制
        handleUpload (file){
            let fileSize = (file.size/1024).toFixed(0)
            if(fileSize > 2048){
                this.$Message.info('文件过大，请上传2M以下的图片')
                return false
            }
            let type = 'image/png,image/jpg,image/jpeg'
            if(!type.includes(file.type)){
                this.$Message.info('仅支持jpg,png格式的图片')
                return false
            }
        },
        handleSuccess (res){
            this.coverPhoto = true
            this.articleData.Cover = res.Data.url
        },
        //拦截编辑器默认的图片上传以及重定义
        handleQuillSuccess (res){   
            let quill = this.$refs.myQuillEditor.quill
            if (res) {
                let length = quill.getSelection().index
                //插入图片链接
                quill.insertEmbed(length, 'image', res.Data.url)  
                //光标移后一位  
                quill.setSelection(length + 1)                  
            } else {
                this.$Message.error('图片插入失败')
            }
        },
        handleChange (value){
            let cateId = value[value.length - 1] 
            //拿到当前选择的栏目id
            this.articleData.CategoryId = cateId
            //根据栏目id获取该分类下的扩展属性对象 
            this.getColumnObjList(cateId)        
        },
        //选择日期
        handlechangeDate (date){
            this.articleData.PublishTime = date
        },
        async getColumnObjList (id){
            let res = await getColumnObject(id)

            if (res.data.Status == 1) this.moduleData = res.data.Data || []
            else{
                this.$Message.info(res.data.Message)
            }
        },
        handleError (){},
        handleMaxSize (){},
        deleteCoverPhoto (){
            this.coverPhoto = false
            this.articleData.Cover = ''
        },
        formatTime (time){
            let newTime = new Date ( time )
            let targetTime = Date.parse(newTime).toString().substr(0,10)

            return targetTime
        },
        submit (){
            let targetData = JSON.parse(JSON.stringify(this.articleData))
            targetData.objData = JSON.stringify( this.articleData.objData )
            targetData.AppId = this.appId
            targetData.CategoryId = this.articleData.CategoryId || ''
            //格式化时间
            targetData.PublishTime = this.formatTime(this.articleData.PublishTime)
            
            if( !this.articleData.Content){
                this.$Message.info("正文内容不可为空")
            }else if( !this.articleData.Title){
                this.$Message.info("标题不可为空")
            }else if( !this.articleData.CategoryId){
                this.$Message.info("请选择栏目")
            }else if( this.articleData.PublishTime.toString() === 'NaN'){
                this.$Message.info("请选择发布时间")
            }else{
                saveArticle(targetData).then(res=>{
                    if(res.data.Status == 1){
                        this.$Message.info('发布成功')
                        this.close()
                    }else{
                        this.$Message.info(res.data.Message)
                        if(res.data.Message === '扩展对象数据不能为空'){
                            this.objData_error_info = true
                        }
                    }
                })
            }
        },
        close (){
            this.$emit('update:show',false)
            this.coverPhoto = false
            this.columnPath = []
            this.moduleData = []
            Object.assign(this.$data.articleData,this.$options.data().articleData)
        },
        //递归处理树形结构的字段信息，将Title、Children、Id换成组件所需要的label、children、value
        convertTree (tree){
            let result = []
            tree.forEach( item => {
                let { Title:label,Children:children,Id:value } = item
                children ? children = this.convertTree(children) : children = ''
                result.push({ label, children, value });
            })
            return result
        },
        //获取文章所属栏目的完整路径
        getColumnPath (id,type,data){
            let r,arr = []
            let res = getPathByKey( id, type, data)
            if(res){
                res.forEach(item =>{
                    arr.push(item[type])
                })
                r = arr.filter(function( ele, index, self){
                    return self.indexOf( ele ) == index
                })
                return r
            }else{
                console.error(res)
            }
        },
        async getArticle (id){
            let res = await getArticleInfo(id)

            if (res.data.Status == 1){
                let time = new Date( res.data.Data.PublishTime * 1000),
                    obj = res.data.Data
                    //格式化时间
                    time = time.toLocaleDateString().replace(/\//g, "-") + " " + time.toTimeString().substr(0, 8)   
                    this.articleData = obj
                    this.articleData.PublishTime = time
                    this.articleData.objData = obj.ObjectData || {}
                    if (this.articleData.Cover) this.coverPhoto = true
                    this.getColumnObjList(obj.CategoryId)

                    this.columnPath = this.getColumnPath(obj.CategoryId,'value',this.selectList)
            }else{
                this.$Message.info(res.data.Message)
            }
        }
    },

    mounted (){
        //加载quill组件图标提示
        addQuillTitle()
    },
    destroyed (){

    }
}
</script>
<style lang="less" scoped>
.slide-in{
    width:100%;
    height:100%;
    background:#fff;
    opacity:1;
    visibility:visible;
    position: absolute;
    top:0px;
    right:0px;
    bottom: 0px;
    box-shadow: 0 0 3px #d5d5d5;
    z-index: 1000;
    overflow: hidden;
    transition: all 0.5s;
    .title{
        width:calc(~"100% - 200px");
        position: fixed;
        top:0;
        padding: 20px;
        line-height: 18px;
        font-size: 1rem;
        border-bottom: 1px solid #d5d5de;
        text-align: center;
        background-color: #fff;
    }
    .article-content{
        margin-top:60px;
        width:calc(~"100% - 200px");
        height:calc(~"100% - 125px");
        overflow-x: hidden;
        overflow-y: auto;
        padding-right: 15px;
        .ivu-form{
            .ivu-form-item{
                width:100%;
                display: inline-block;
                margin-bottom: 15px;
                padding-left:150px;
                /deep/.ivu-form-item-label{
                    font-size:14px;
                }
                /deep/.ivu-form-item-content{
                    .ivu-upload{
                        text-align: left;
                    }
                    .ivu-form-item-error-tip{
                        margin-left:35px;
                    }
                    .ivu-cascader .ivu-cascader-menu{
                        min-width: 230px;
                    }
                    .ivu-input-wrapper{
                        >input{
                            font-size:14px;
                            border-radius: 0;
                        }
                        >textarea{
                            border-radius: 0;
                        }
                    }
                    /deep/.ivu-form-item-error-tip{
                        margin-left:0;
                        padding:2px 0;
                    }
                }
                /deep/.ql-container .ql-editor{height: 300px;overflow-y: auto;}

            }
        }
    }
    .info-footer{
    position: fixed;
    bottom:0px;
    width: calc(~"100% - 200px");
    border-top:1px solid #d5d5de;
    background-color: #fff;
    height: 65px;
    line-height: 65px;
    text-align: center;
    .ant-btn{
        display: inline-block;
        margin-bottom: 0;
        font-weight: 500;
        text-align: center;
        touch-action: manipulation;
        cursor: pointer;
        background-image: none;
        border: 1px solid transparent;
        white-space: nowrap;
        line-height: 1.15;
        padding: 0 15px;
        font-size: 12px;
        height: 28px;
        user-select: none;
        transition: all 0.3s cubic-bezier(0.645, 0.045, 0.355, 1);
        position: relative;
    }
    .ant-btn-primary{
        color: #fff;
        background-color: #108ee9;
        border-color: #108ee9;
        margin-right:20px;
    }
    .ant-btn-default{
        background-color:#fff;
        border:1px solid #108ee9;
    }

}
.close{
    position: fixed;
    top:-10px;
    z-index:1001;
    color:#d5d5d5;
    transition: all 0.5s;
    cursor: pointer;
}
.close:hover{
    color:#fff;
}
.closeShow{
    left:calc(~"134px");
}
.closeHide{
    left:100%;
}
}
.show{
    position: fixed;
    left:calc(~"200px");
}
.hide{
    position: fixed;
    left:100%;
}

.must-symbol{
    color:red;
    font-size:12px;
    margin-right:4px;
}
.noNecessary{
    width:0;
}
.article-attribute{
    position:relative;
    margin-bottom:10px;
    left:17px;
    margin-top:25px;
    .extend-span{
        width:4px;
        font-weight: 400;
        background:#108cee;
        padding:8px 0;
        float: left;
    }
    .extend-nature{
        width:85%;
        text-align: left;
        margin:0 auto;
        font-weight: bold;
        display: inline-block;
        line-height: 16px;
        margin-left:14px;
        font-size: 1rem;
    }

}
</style>


