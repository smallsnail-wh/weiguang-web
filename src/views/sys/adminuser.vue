<template>
	<div style="margin: 20px;">
        <div>
            <Row style="margin-bottom: 25px;">
                <Col span="8">登录名：
                	<Input v-model="loginName" placeholder="请输入..." style="width:200px"></Input>
                </Col>
                <Col span="8"><Button type="primary" shape="circle" icon="ios-search" @click="search()">搜索</Button></Col>
            </Row>
        </div>            
        <div>
            <ul>
                <li>
                    <Button type="primary" icon="plus-round" @click="openNewModal()">新建</Button>
                    <Button type="error" icon="trash-a" @click="del()">删除</Button>
                </li>
                <li>
                    <div style="padding: 10px 0;">
                    	<Table border :columns="columns1" :data="data1" :height="400" @on-selection-change="s=>{change(s)}"></Table>
                    </div> 
                </li>
                <li>
                    <div style="text-align: right;">
                        <Page :total="total" :page-size="pageInfo.pageSize" show-elevator show-total @on-change="e=>{pageSearch(e)}"></Page>
                    </div>  
                </li>
            </ul>
        </div>
        <!--添加modal-->  
        <Modal :mask-closable="false" :visible.sync="newModal" :loading = "loading" v-model="newModal" width="600" title="新建" @on-ok="newOk('userNew')" @on-cancel="cancel()">
            <Form ref="userNew" :model="userNew" :rules="ruleNew" :label-width="80" >
                <Row>
                    <Col span="12">
                        <Form-item label="用户名:" prop="name">
                            <Input v-model="userNew.name" style="width: 204px"/>
                        </Form-item>
                        <Form-item label="手机号:" prop="mobile">
                            <Input v-model="userNew.mobile" style="width: 204px"/>
                        </Form-item>
                    </Col>
                </Row>
                <Row>
                    <Col span="12">
                        <Form-item label="密码:" prop="password">
                            <Input v-model="userNew.password" type="password" style="width: 204px"/>
                        </Form-item>
                    </Col>
                    <Col span="12">
                        <Form-item label="确认密码:" prop="passwordAgain">
                            <Input v-model="userNew.passwordAgain" type="password" style="width: 204px"/>
                        </Form-item>
                    </Col>
                </Row>
               
            </Form>
        </Modal>
        <!--修改modal-->  
        <Modal :mask-closable="false" :visible.sync="modifyModal" :loading = "loading" v-model="modifyModal" width="600" title="修改" @on-ok="modifyOk('userModify')" @on-cancel="cancel()">
             <Form ref="userModify" :model="userModify" :rules="ruleModify" :label-width="80" >
                <Row>
                    <Col span="12">
                        <Form-item label="登录名:" prop="loginName">
                            <Input v-model="userModify.loginName" style="width: 204px"/>
                        </Form-item>
                    </Col>
                    <Col span="12">
                        <Form-item label="用户名:" prop="name">
                            <Input v-model="userModify.name" style="width: 204px"/>
                        </Form-item>
                    </Col>
                </Row>
                <Row>
                    <Col span="12">
                        <Form-item label="密码:" prop="password">
                            <Input v-model="userModify.password" type="password" style="width: 204px"/>
                        </Form-item>
                    </Col>
                </Row>
                <Row>
                    <Col span="12">
                        <Form-item label="邮箱:" prop="email">
                            <Input v-model="userModify.email" style="width: 204px"/>
                        </Form-item>
                    </Col>
                </Row>
            </Form>
        </Modal>
        <!--配置角色modal-->  
        <Modal v-model="roleModal" width="500" title="角色配置" @on-ok="roleOk()" @on-cancel="cancel()">
            <div>
                <Table border :columns="columns2" :data="data2" :height="260"  @on-selection-change="s=>{change2(s)}"></Table>
            </div>
        </Modal>
    </div>
</template>
<script>
	export default {
        data () {
            return {
                customerTypeList:[
                    {
                        value:'0',
                        label:'普通用户'

                    },
                    {
                        value:'1',
                        label:'全职销售'

                    }
                ],
                customerType:null,
                /*用于查找的登录名*/
                loginName:null,
            	/*选择的数量*/
                count:null,
            	/*选中的组数据*/
                groupId:null,
            	/*新建modal的显示参数*/
                newModal:false,
                /*修改modal的显示参数*/
                modifyModal:false,
                /*角色配置modal的显示参数*/
                roleModal:false,
            	/*分页total属性绑定值*/
                total:0,
                /*loading*/
                loading: true,
                /*pageInfo实体*/
                pageInfo:{
                	page:0,
                	pageSize:10
                },
                /*user实体*/
                user:{
                    id:null,
                    name:null,
                    loginName:null,
                    password:null,
                    passwordAgain:null,
                    email:null
                },
                /*用于添加的user实体*/
                userNew:{
					name:null,
                    mobile:null,
					password:null,
                    passwordAgain:null,
                    customerType:null
                },
                /*用于修改的user实体*/
                userModify:{
                	id:null,
					name:null,
					loginName:null,
					password:null,
					email:null
                },
                /*新建验证*/
                ruleNew:{
                    name: [
                        { type:'string',required: true, message: '输入用户名', trigger: 'blur' }
                    ],
                    mobile: [
                        { type:'string',required: true, message: '输入手机号', trigger: 'blur' }
                    ],
                    password: [
                        { type:'string',required: true, message: '输入密码', trigger: 'blur' }
                    ],
                    passwordAgain: [
                        { type:'string',required: true, message: '输入再次密码', trigger: 'blur' }
                    ]
                },
                /*修改验证*/
                ruleModify:{
                    name: [
                        { type:'string',required: true, message: '输入用户名', trigger: 'blur' }
                    ],
                    loginName: [
                        { type:'string',required: true, message: '输入登录名', trigger: 'blur' }
                    ],
                    password: [
                        { type:'string',required: true, message: '输入密码', trigger: 'blur' }
                    ],
                    email: [
                        { required: true, message: '输入邮箱', trigger: 'blur' },
                        { type:'email', message: '输入正确的邮箱格式', trigger: 'blur' }
                    ]
                },
            	/*表显示字段*/
            	columns1: [
                    {
                        type: 'selection',
                        width: 60,
                        align: 'center'
                    },
                    {
                        title: '用户名',
                        key: 'name'
                    },
                    {
                        title: '手机号',
                        key: 'mobile'
                    },
                    {
                        title: '账号余额',
                        key: 'money'
                    }
                ],
                /*表数据*/
                data1: [],
                /*表显示字段*/
                columns2: [
                    {
                        type: 'selection',
                        width: 60,
                        align: 'center'
                    },
                    {
                        title: '角色名称',
                        width: 120,
                        key: 'name'
                    },
                    {
                        title: '描述',
                        key: 'describe'
                    }
                ],
                /*表数据*/
                data2:[],
                /*data2的临时存储*/
                data2Temp:[],
                /*用户与角色关系列表*/
                relationList:null
            }
        },
        mounted(){
        	/*页面初始化调用方法*/
            this.getTable({
                "pageInfo":this.pageInfo,
                "loginName":this.loginName
            });
            this.axios({
              method: 'get',
              url: '/roles/all'
            }).then(function (response) {
                this.data2Temp = response.data;
            }.bind(this)).catch(function (error) {
              alert(error);
            });
        },
        methods:{
        	/*pageInfo实体初始化*/
        	initPageInfo(){
        		this.pageInfo.page = 0;
        		this.pageInfo.pageSize = 10;
        	},
            /*user实体初始化*/
            initUser(){
                this.user.id = null;
                this.user.name = null;
                this.user.loginName = null;
                this.user.password = null;
                this.user.email = null;
            },
            /*userNew实体初始化*/
            initUserNew(){
                
                this.userNew.name = null;
                this.userNew.mobile = null;
                this.userNew.password = null;
                this.userNew.passwordAgain = null;
                this.userNew.customerType = null;
            },
            /*userModify实体初始化*/
            initUserModify(){
                this.userModify.id = null;
                this.userModify.name = null;
                this.userModify.loginName = null;
                this.userModify.password = null;
                this.userModify.email = null;
            },
            /*userNew设置*/
            userSet(e){
                this.user.id = e.id;
                this.user.name = e.name;
                this.user.loginName = e.loginName;
                this.user.password = e.password;
                this.user.email = e.email;
            },
            /*userNew设置*/
            userNewSet(e){

                this.userNew.name = e.name;
                this.userNew.mobile = e.mobile;
                this.userNew.password = e.password;
                this.userNew.passwordAgain = e.password;
                this.userNew.customerType = e.customerType;
            },
            /*userModify设置*/
            userModifySet(e){
                this.userModify.id = e.id;
                this.userModify.name = e.name;
                this.userModify.loginName = e.loginName;
                this.userModify.password = e.password;
                this.userModify.email = e.email;
            },
            /*得到表数据*/
            getTable(e) {
                this.axios({
                  method: 'get',
                  url: '/admin/adminusers',
                  params: {
                    'page':e.pageInfo.page,
                    'pageSize':e.pageInfo.pageSize,
                    'loginName':e.loginName
                  }
                }).then(function (response) {
                    this.data1=response.data.data;
                    this.total=response.data.totalCount;
                }.bind(this)).catch(function (error) {
                  alert(error);
                });
            },
            /*搜索按钮点击事件*/
            search(){
                this.initPageInfo();
                this.getTable({
                    "pageInfo":this.pageInfo,
                    "loginName":this.loginName
                });   
            },
            /*分页点击事件*/
            pageSearch(e){
                this.pageInfo.page = e-1;
                this.getTable({  
                    "pageInfo":this.pageInfo,
                    "loginName":this.loginName
                });
            },
            /*新建点击触发事件*/
            openNewModal(){
                this.newModal = true;
                this.initUserNew();
                this.data1.sort();
                this.count = 0;
                this.groupId = null;
            },
            /*新建modal的newOk点击事件*/
            newOk (userNew) { 
                this.$refs[userNew].validate((valid) => {
                    if (valid) {
                        if(this.userNew.password == this.userNew.passwordAgain){
                           /* this.initUser();*/
                            /*this.userSet(this.userNew);*/
                            this.userNew.customerType = 3;
                            this.axios({
                                method: 'post',
                                url: '/admin/users/user',
                                data: this.userNew
                            }).then(function (response) {
                                this.initUserNew();
                                this.getTable({
                                    "pageInfo":this.pageInfo,
                                    "loginName":this.loginName
                                });
                                this.$Message.info('新建成功');
                            }.bind(this)).catch(function (error) {
                                alert(error);
                            });  
                            this.newModal = false;
                        }else{
                            this.$Message.error('两次输入的密码不相同！');
                            this.loading = false;
                            this.$nextTick(() => {
                                this.loading = true;
                            });
                        }
                    }else {
                        this.$Message.error('表单验证失败!');
                        setTimeout(() => {
                            this.loading = false;
                            this.$nextTick(() => {
                                this.loading = true;
                            });
                        }, 1000);
                    }
                })
            },
            /*点击修改时判断是否只选择一个*/
            openModifyModal(){
                if(this.count > 1 || this.count < 1){
                    this.modifyModal= false; 
                    this.$Message.warning('请至少选择一项(且只能选择一项)');  
                }else{
                    this.modifyModal = true;
                }
            },
            /*修改modal的modifyOk点击事件*/
             modifyOk (userModify) { 
                this.$refs[userModify].validate((valid) => {
                    if (valid) {
                        this.initUser();
                        this.userSet(this.userModify);
                        this.axios({
                          method: 'put',
                          url: '/users/'+this.user.id,
                          data: this.user
                        }).then(function (response) {
                            this.initUserNew();
                            this.getTable({
                                "pageInfo":this.pageInfo,
                                "loginName":this.loginName
                            });
                            this.$Message.info('修改成功');
                        }.bind(this)).catch(function (error) {
                          alert(error);
                        });  
                        this.modifyModal = false;
                    }else {
                        this.$Message.error('表单验证失败!');
                        setTimeout(() => {
                            this.loading = false;
                            this.$nextTick(() => {
                                this.loading = true;
                            });
                        }, 1000);
                    }
                })
            },
            /*modal的cancel点击事件*/
            cancel () {
                this.$Message.info('点击了取消');
            },
            /*table选择后触发事件*/
            change(e){
                if(e.length==1){
                    this.userModifySet(e['0']);
                }
                this.setGroupId(e);              
            },
            /*通过选中的行设置groupId的值*/
            setGroupId(e){
                this.groupId=[];
                this.count=e.length;
                for (var i = 0; i <= e.length - 1; i++) {
                    this.groupId.push(e[i].id);
                }
            },
            /*删除table中选中的数据*/
            del(){
                if(this.groupId!=null && this.groupId!=""){
                    this.axios({
                      method: 'delete',
                      url: '/admin/adminusers',
                      data: this.groupId
                    }).then(function (response) {
                        this.getTable({
                            "pageInfo":this.pageInfo,
                            "loginName":this.loginName
                        });
                        this.groupId=null;
                        this.count=0;
                        this.$Message.info('删除成功');
                    }.bind(this)).catch(function (error) {
                        alert(error);
                    });
                }
            },
            /*表格中双击事件*/
            dblclick(e){
                this.userModifySet(e);
                this.modifyModal = true;
                this.data1.sort();
            },
            /*流程配置*/
            relationSet(e){
                this.roleModal = true;
                this.data2 = [];
                this.axios({
                  method: 'get',
                  url: '/relations/'+e.id
                }).then(function (response) {
                    var roleList = [];
                    for(var i in response.data){
                        roleList.push(response.data[i].roleId);
                    }
                    for(var i in this.data2Temp){
                        if(roleList.indexOf(this.data2Temp[i].id) == -1){
                            this.data2.push({
                                "id": this.data2Temp[i].id,
                                "name": this.data2Temp[i].name,
                                "describe": this.data2Temp[i].describe,
                                "userId": e.id,
                                "_checked": false
                            });
                        }else {
                            this.data2.push({
                                "id": this.data2Temp[i].id,
                                "name": this.data2Temp[i].name,
                                "describe": this.data2Temp[i].describe,
                                "userId": e.id,
                                "_checked": true
                            });
                        }
                    }   
                }.bind(this)).catch(function (error) {
                  alert(error);
                });
            },
            /*配置角色modal确认按钮点击事件*/
            roleOk(){
                if(this.relationList!=null){
                    this.axios({
                      method: 'post',
                      url: '/relations',
                      data: this.relationList
                    }).then(function (response) {
                        this.$Message.info('配置成功'); 
                    }.bind(this)).catch(function (error) {
                      alert(error);
                    });
                    this.relationList = null;
                }
            },
            /*配置角色modal中表选择改变事件*/
            change2(e){
                this.relationList = [];
                if(e.length == 0){
                    this.relationList.push({
                        "userId": this.data2[0].userId
                    }); 
                }
                for (var i in e) {
                    this.relationList.push({
                        "userId": e[i].userId,
                        "roleId": e[i].id
                    });  
                }
            }
        }
    }
</script>