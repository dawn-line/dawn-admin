﻿<template>
<yl-panelpage :titleName="'用户管理'" >
<div slot="content">
  <yl-columnlay>
              <div slot="left"  >
                   <yl-organizeTreeWithPanel :reloadIsShow="true" v-model="origanizeWithPanel" @getCurrentNode="_getCrureentNode"  :placeholder="'请选择'"   style="width:200px"> </yl-organizeTreeWithPanel>   
                </div>
                <div slot="right" >
                    <yl-layout>
                        <div slot="fristbox" >
                          <yl-toolbar>
                                <el-form  :inline="true"  >
                                        <el-form-item class="form-content-vertical">
                                       <el-input  placeholder="用户账号" size="small" style="width:200px" v-model="searchModel.userName"></el-input> 
                                        </el-form-item>
                                         <el-form-item class="form-content-vertical">
                                       <el-input  placeholder="用户姓名" size="small" style="width:200px" v-model="searchModel.realName"></el-input> 
                                        </el-form-item>
                                        <el-form-item class="form-content-vertical">
                                            <el-button type="primary" icon="el-icon-search" size="small"  @click="_reload">查 询</el-button>
                                        </el-form-item>
                                </el-form>
                          </yl-toolbar>
                        <yl-toolbar>
                            <el-button-group>
                                <el-button type="primary" icon="el-icon-plus" size="small"  @click="_add" >添加</el-button>
                                <el-button type="primary" icon="el-icon-edit"  size="small" :disabled="selectRows.length===0" @click="_edit">编辑</el-button>
                                <el-button type="danger"  icon="el-icon-delete" size="small" v-permissionSetting="'Users.Delete'" :disabled="selectRows.length===0" @click="_delete" >删除</el-button>
                                <el-button type="warning"  icon="el-icon-setting" size="small"  :disabled="selectRows.length===0" @click="_resetPass" >重置密码</el-button>
                            </el-button-group>
                        </yl-toolbar>
                        </div>
                         <div slot="secondbox" class="flexbox">
                                <yl-table ref="table"
                                    @reload="_reload"
                                    @current-change="_currentChange"
                                    :configs="tableConfig" 
                                    :input="mainInput.inputModel"
                                    :tableloading="mainTableLoading" 
                                    >
                                        <template slot="isActive" scope="scope">
                                            <el-tag type="primary" v-if="scope.row.isActive">启用</el-tag>
                                            <el-tag type="danger" v-else>禁用</el-tag>
                                        </template>
                                       
                                </yl-table>
                         </div>
                            </yl-layout>
                     </div>
     </yl-columnlay>
    <!--编辑界面-->
         <el-dialog ref="mainDialog" 
                    title="编辑" 
                    :visible.sync="addFormVisible"
                    top="5%" 
                    width="50%"
                    :modal-append-to-body="false" 
                    :close-on-click-modal="false"  >
                    <Edit :selectRow="currentRows" @close="_close"  v-if="addFormVisible"></Edit>
         </el-dialog>
 </div>
</yl-panelpage>
</template>
<script type="text/babel">
import Edit from './UserEdit';
import util from '@/common/js/util';
import  {inputModel} from '@/api/inputmodel';
import organizeTreeWithPanel from '@/ocomponents/organize/organizeTreeWithPanel'

import {
            requestGetUserPageList,
            requestDeleteUser,
            requestGetUserForEdit,
            requestCreateOrUpdateUser
        }   from '@/api/user';

export default {
    data(){
        return {
             tableData: {},
                mainTableLoading:false,
                selectRows:[],
                currentRows:{},
                origanizeWithPanel:[],
                searchModel:{
                     userName:'',
                     realName:'',  
                    },
                addFormVisible:false,
                mainInput:new inputModel(),
               
                selectNode:{id:"00000000-0000-0000-0000-000000000000",code:''},
                defaultProps: {
                        children: 'children',
                        label: 'text',
                        id: 'id'
                    },
            }
    },
     computed: {
            tableConfig: {
                    get () {
                        return {
                            table: {
                                attr: {
                                    data: this.tableData,
                                    highlightCurrent:true,
                                }
                            },
                            columns: [
                                {attr: { type: 'index',label: '序号', width:50,align: 'center',headerAlign:'center' }},
                                //{attr: { prop: 'orgId', label: 'orgid', width:120,  } },   
                                {attr: { prop: 'orgFullPathName', label: '所在部门', width:180,  } },  
                                 {attr: { prop: 'isActive', label: '状态', width:80, scopedSlot: 'isActive',} },
                                {attr: { prop: 'realName', label: '姓名', width:100,  } },   
                                {attr: { prop: 'userName', label: '用户名', width:100,  } },   
                                //{attr: { prop: 'password', label: '密码', width:120,  } },   
                                {attr: { prop: 'gender', label: '性别', width:80,  } },   
                                {attr: { prop: 'telephoneNo', label: '电话', width:130,  } },   
                                {attr: { prop: 'emailAddress', label: '电子邮件', width:180,  } }, 
                                {attr: { prop: 'sortCode', label: '排序',} },
                            ]
                        }
                    }
                }
        },
    methods:{
        _resetPass(){
            let params={};
            params.id=this.selectRows[0].id;
            requestGetUserForEdit(params).then(data=>{
                let para=data.result;
                para.user.password='B9E6E1E515598CA5'
                requestCreateOrUpdateUser(para).then(data1=>{
                    if(data1.success){
                        this.$notify({
                                title: '成功',
                                message: '重置密码成功！',
                                type: 'success'
                                });
                    }else{
                        this.$notify.error({
                                title: '错误',
                                message: '重置密码失败！',
                                });
                    }
                })
            })
        },
        _getUserPageList(){
                this.selectRows=[];
                this.mainTableLoading=true; //开启加载
                var inputArr=[ 
                               {key:"UserName",op:"LIKE",value:this.searchModel.userName},
                               {key:"RealName",op:"LIKE",value:this.searchModel.realName},
                               {key:"OrgCode",op:"LIKEL",value:this.selectNode.code},
                            //    {key:"UserType",op:"EQ",value:0},
                               {key:"IsStatic",op:"EQ",value:false}
                             ];
                this.mainInput.addqueryConditionItem(inputArr);
                requestGetUserPageList(this.mainInput.inputModel).then(data =>{
                            if(data.success){
                                this.tableData=data.result;
                                }
                            else {
                                this.$message.error('失败！'+data.error.message);
                            }
                          this.mainTableLoading=false;
                }).catch(function(error){
                       this.mainTableLoading=false;
                });
        },
         _handleNodeClick(result, resolve) {  
                 //点击加载 
                  this.selectNode=result;
                  this._reload();
            },
        _currentChange(val){
                //单选时的方法
                if(val!=null){
                     this.selectRows=[];
                     this.selectRows.push(val);
                }
            },
      _getCrureentNode(node){
            this.origanizeWithPanel=node.text;
            this.selectNode.id=node.id;
            this.selectNode.code=node.code;
            this._reload();
        },
        _selectionChange(val){
            //多选时的方法
            if(val!=null){
                this.selectRows=[];
                this.selectRows=this.selectRows.concat(val);
            }
        },
        toggleRowSelection (row) {
            this.$refs.table.toggleRowSelection(row)
        },
        _reload(){
            this._getUserPageList();
        },
        _add(){
            this.addFormVisible=true;
            this.currentRows={};
        },
        _edit(){
                this.addFormVisible=true;
                this.currentRows=this.selectRows[0];
        },
        _delete(){
            this.$confirm('确认要删除所选项目吗?', '提示', {
                type: 'warning'
            }).then(() => {
                var obj={}; 
                obj.id=this.selectRows[0].id;
                requestDeleteUser(obj).then(data =>{
                        if(data.success){
                                this._reload();
                                this.$notify({
                                    title: '成功',
                                    message: '删除数据成功！',
                                    type: 'success'
                                    });
                            }
                            else {
                                this.$notify.error({
                                    title: '错误',
                                    message: '删除数据失败！'+data.error.message,
                                    });
                            }
                });
            })
        },
        _close(){
                this.addFormVisible=false;
                this._reload();
        },
    },
    components:{
         Edit,
        'yl-organizeTreeWithPanel':organizeTreeWithPanel,

        },
    mounted(){
         if(this.getUserInfo().user.orgCode!=undefined){
             this.selectNode.code=this.getUserInfo().user.orgCode;
         }else{
             this.selectNode.code='00001'
         }
         
        this._reload();
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus" rel="stylesheet/stylus" scoped>
</style>

