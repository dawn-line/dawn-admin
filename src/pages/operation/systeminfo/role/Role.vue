﻿<template>
    <yl-panelpage :titleName="'角色管理'" >
        <div slot="content"> 
         <yl-columnlay>
            <div slot="left"  >
                    <yl-panelpage 
                        isZone 
                        :helpIsShow="false" 
                        :treeLoading="treeLoading"  
                        :titleName="'角色树'" 
                        :icon="'icon-tree'" 
                        @reload="_treeReload" 
                        :reloadIsShow="false"
                        
                        :widthNum="'210px'" >
                        <div slot="content">
                            <el-tree 
                                    :data="roleTreeData" 
                                    :expand-on-click-node="false" 
                                    highlight-current
                                    default-expand-all  
                                    :render-content="renderContent"
                                    :props="defaultProps"
                                    @node-click="_handleNodeClick" 
                                    style="height:100%">
                            </el-tree>
                        </div>
                    </yl-panelpage>
                </div>
             <div slot="right" >
                 <yl-layout >
                <div slot="fristbox" >
                    <yl-toolbar>
                        <el-form  :inline="true"  >
                            <el-form-item class="form-content-vertical">
                                <el-input  placeholder="角色名称" size="small" style="width:200px" v-model="searchModel.displayName"></el-input>  
                            </el-form-item>
                            <el-form-item class="form-content-vertical">
                                <el-button type="primary" icon="el-icon-search" size="small"  @click="_reload">查 询</el-button>
                            </el-form-item>
                        </el-form>
                    </yl-toolbar>
                    <yl-toolbar>
                        <el-button-group>
                            <el-button type="primary" icon="el-icon-plus" size="small" v-permissionSetting="'Roles.Create'"  @click="_add" >添加</el-button>
                            <el-button type="primary" icon="el-icon-edit"  size="small" v-permissionSetting="'Roles.Edit'" :disabled="selectRows.length===0" @click="_edit">编辑</el-button>
                            <el-button type="danger"  icon="el-icon-delete" size="small" v-permissionSetting="'Roles.Delete'" :disabled="selectRows.length===0" @click="_delete" >删除</el-button>
                            <!--<el-button type="success"  icon="plus" size="small" v-permissionSetting="'show'"  @click="_exportExecl" >导出execl</el-button>-->
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
                        <!--自定义列模板
                                <template slot="isDisable" scope="scope">
                                    <el-tag type="primary" v-if="scope.row.showFlag">启用</el-tag>
                                    <el-tag type="danger" v-else>禁用</el-tag>
                                </template>
                                -->
                    </yl-table>
                </div>
                </yl-layout>
                </div>
         </yl-columnlay>
            <!--编辑界面-->
                <el-dialog ref="mainDialog" 
                    title="编辑" 
                    size="small"  
                    top="5%" 
                    width='45%'
                    :visible.sync="addFormVisible"
                    :modal-append-to-body="false" 
                    :close-on-click-modal="false"  >
                 <Edit :selectRow="currentRows" 
                        @close="_close"  
                        v-if="addFormVisible"></Edit>
            </el-dialog>
        </div>
    </yl-panelpage>
</template>

<script type="text/babel">
import Edit from './RoleEdit';
import  {inputModel} from '@/api/inputmodel';
import treeMixn from '@/mixns/tree.js';
import {
            requestGetRolePageList,
            requestDeleteRole,
            requestGetRoleTreeList
        }   from '@/api/role';
import util from '@/common/js/util';
export default {
    mixins: [treeMixn],
    data(){
        return {
             tableData: {},
                mainTableLoading:false,
                selectRows:[],
                currentRows:{},
                searchModel:{
                      displayName:'',  
                    },
                addFormVisible:false,
                mainInput:new inputModel(),
                roleTreeData:[],
                defaultProps: {
                    children: 'children',
                    label: 'text',
                    id: 'id'
                },
                selectNode:{id:this.guidOfNull()},
                treeLoading:false,
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
                                {attr: { type: 'index',label: '序号', width:50,align: 'center',headerAlign:'center'  }},
                                {attr: { prop: 'appCode', label: '应用代码', width:110,  } },   
                                {attr: { prop: 'name', label: '名称', width:200,  } },   
                                {attr: { prop: 'displayName', label: '显示名称', width:200,  } },   
                                {attr: { prop: 'sortCode', label: '排序'} },
                            ]
                        }
                    }
                }
        },
    methods:{
        _getRolePageList(){
                this.selectRows=[];
                this.mainTableLoading=true; //开启加载
                var inputArr=[ 
                              {key:"DisplayName",op:"LIKE",value:this.searchModel.displayName},
                              {key:"ParentId",op:"EQ",value:this.selectNode.id},  
                              {key:"IsStatic",op:"EQ",value:false}
                             ];
                this.mainInput.addqueryConditionItem(inputArr);
                requestGetRolePageList(this.mainInput.inputModel).then(data =>{
                            if(data.success){
                                this.tableData=data.result;
                                }
                            else {
                                 this.$message.error('获取数据失败！'+data.error.message);
                            }
                          this.mainTableLoading=false;
                }).catch(function(error){
                       this.mainTableLoading=false;
                });
        },
        _currentChange(val){
                //单选时的方法
                if(val!=null){
                     this.selectRows=[];
                     this.selectRows.push(val);
                }
               
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
            this._getRolePageList();
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
            var inputArr=[{key:"ParentId",op:"EQ",value:this.selectRows[0].id}];
            this.mainInput.inputModel.sorting="ParentId";
            this.mainInput.addqueryConditionItem(inputArr);
              requestGetRolePageList(this.mainInput.inputModel).then(data =>{
               if(data.result.data.length>0){
                   this.$message.error('删除失败！该角色下还有未删除的子节点！');
                   this._reload();
               }else{
                   this.$confirm('确认要删除所选项目吗?', '提示', {
                type: 'warning'
            }).then(() => {
                var obj={}; 
                obj.id=this.selectRows[0].id;
                requestDeleteRole(obj).then(data =>{
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
               }
           })
        },
        _handleNodeClick(result, resolve) {  
                    //点击加载 
                    this.selectNode=result;
                    this._reload();
                }, 
        _close(){
                this.addFormVisible=false;
                this._reload();
        },   
        _getRoleTreeList(){
                let _this=this;
                this.treeLoading=true;
                requestGetRoleTreeList().then(data => {
                    if(data.success){
                        this.roleTreeData=data.result;
                    }
                else {
                    _this.$message.error('获取数据失败！'+data.error.message);
                }
                this.treeLoading=false;
                })
        },
        _treeReload(){
                this._getRoleTreeList();
            }, 
        _exportExecl(){
            const tHeader = ['序号', '应用代码', '名称', '显示名称', '排序'];
            const filterVal = ['id', 'appCode', 'name', 'displayName', 'sortCode'];
            util.exportExecl(tHeader,filterVal,this.tableData.data,'列表excel');
        }
    },
    components:{
         Edit,
        },
    mounted(){
        this._reload();
        this._getRoleTreeList();
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus" rel="stylesheet/stylus" scoped>
</style>

