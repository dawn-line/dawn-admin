﻿<template>
    <yl-panelpage titleName="字典管理" >
        <div slot="content">
            <yl-columnlay>
                <div slot="left"  >
                    <yl-panelpage isZone
                     :helpIsShow="false" 
                     :treeLoading="treeLoading"  
                      :titleName="'系统字典'" @reload="_treeReload" :reloadIsShow="true" :icon="'icon-tree'" :widthNum="'210px'" >
                        <div slot="content">
                            <el-tree :data="dictionaryTreeData"  
                                        :props="defaultProps"
                                        @node-click="_handleNodeClick"
                                        :highlight-current="true"  
                                        :render-content="renderContent"
                                        :expand-on-click-node="false" 
                                        style="height:100%"></el-tree>
                            </div>
                      </yl-panelpage>
                </div>
                <div slot="right" >
                    <yl-layout>
                            <div slot="fristbox" >
                                <yl-toolbar>
                                        <el-form  :inline="true"  >
                                                <el-form-item class="form-content-vertical">
                                                    <el-input  placeholder="字典名称" size="small" style="width:200px" v-model="searchModel.fullName"></el-input> 
                                                </el-form-item>
                                                <el-form-item class="form-content-vertical">
                                                    <el-button type="primary" icon="el-icon-search" size="small"  @click="_reload">查 询</el-button>
                                                </el-form-item>
                                        </el-form>
                                </yl-toolbar>
                                <yl-toolbar>
                                    <el-button-group>
                                            <el-button type="primary" v-permissionSetting="'DataDictionarys.Create'" icon="el-icon-plus" size="small"  @click="_add" >添加</el-button>
                                            <el-button type="primary" v-permissionSetting="'DataDictionarys.Edit'" icon="el-icon-edit"  size="small" :disabled="selectRows.length===0" @click="_edit">编辑</el-button>
                                            <el-button type="danger" v-permissionSetting="'DataDictionarys.Delete'"  icon="el-icon-delete" size="small"  :disabled="selectRows.length===0" @click="_delete" >删除</el-button>
                                        </el-button-group>
                                </yl-toolbar>
                            </div>
                            <div slot="secondbox"  class="flexbox">
                                    <yl-table ref="table"
                                        @reload="_reload"
                                        @current-change="_currentChange"
                                        :configs="tableConfig" 
                                        :input="mainInput.inputModel"
                                        :tableloading="mainTableLoading" 
                                        >
                                         <template slot="isStatic" scope="scope">
                                                <el-tag type="primary" v-if="scope.row.isStatic==0">否</el-tag>
                                                <el-tag type="danger" v-else>是</el-tag>
                                         </template>
                                           <template slot="isTree" scope="scope">
                                                <el-tag type="primary" v-if="scope.row.isTree==0">否</el-tag>
                                                <el-tag type="danger" v-else>是</el-tag>
                                         </template>
                                           <template slot="isLeaf" scope="scope">
                                                <el-tag type="primary" v-if="scope.row.isLeaf==0">否</el-tag>
                                                <el-tag type="danger" v-else>是</el-tag>
                                         </template>
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
                        <el-dialog ref="mainDialog" title="编辑" :visible.sync="addFormVisible" width="40%"  top="5%" 
                                    :modal-append-to-body="false" :close-on-click-modal="false"  >
                                    <Edit :selectRow="currentRows" @close="_close"  v-if="addFormVisible"></Edit>
                        </el-dialog>
            
        </div>
    </yl-panelpage>
</template>

<script type="text/babel">
import Edit from './DataDictionaryDetailEdit';
import util from '@/common/js/util';
import  {inputModel} from '@/api/inputmodel';
import treeMixn from '@/mixns/tree.js';
import {
            requestGetDataDictionaryDetailPageList,
        }   from '@/api/datadictionarydetail';
import {
            requestGetDataDictionaryTreeList,
            requestDeleteDataDictionary,
            requestGetDataDictionaryPageList
        }   from '@/api/datadictionary';

export default {
    mixins: [treeMixn],
    data(){
        return {
                tableData: {},
                mainTableLoading: false,
                selectRows: [],
                currentRows: {},
                searchModel: {
                      fullName:'',  
                    },
                addFormVisible:false,
                mainInput:new inputModel(),
                defaultProps: {
                        children: 'children',
                        label: 'text',
                        id: 'id'
                    },
                dictionaryTreeData:[],
                selectNode:{id:"00000000-0000-0000-0000-000000000000"},
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
                                {attr: { type: 'index',label: '序号', width:50,align: 'center',headerAlign:"center" }}, 
                                {attr: { prop: 'code', label: '编码', width:150,  } },   
                                {attr: { prop: 'fullName', label: '名称', width:150,  } },  
                                {attr: { prop: 'isStatic', label: '系统参数',  width:120, scopedSlot: 'isStatic',   } }, 
                                {attr: { prop: 'isLeaf', label: '子节点',  width:100, scopedSlot: 'isLeaf',   } }, 
                                {attr: { prop: 'isTree', label: '树结构',  width:120, scopedSlot: 'isTree',   } },  
                                {attr: { prop: 'sortCode', label: '排序',width:80,align: 'center',headerAlign:"center"    } }, 
                                {attr: { prop: 'remark', label: '备注',} },   
                               
                            ]
                        }
                    }
                }
        },
    methods:{    
         _getGetDataDictionaryTreeList(){
                let _this=this;
                this.treeLoading=true;
                requestGetDataDictionaryTreeList().then(data => {
                        if(data.success){
                            this.dictionaryTreeData=util.returnDatabyTree(data.result,'00000000-0000-0000-0000-000000000000');
                        }
                        else {
                            _this.$message.error('获取数据失败！');
                        }
                        this.treeLoading=false;
                    })
            },
        _handleNodeClick(result, resolve) {  
                 //点击加载 
                  this.selectNode=result;
                  this._reload();
            },
        _getDataDictionaryDetailPageList(){
                let _this=this;
                this.selectRows=[];
                this.mainTableLoading=true; //开启加载
                var inputArr=[ 
                               {key:"FullName",op:"LIKE",value:this.searchModel.fullName},
                               {key:"ParentId",op:"EQ",value:this.selectNode.id},
                               {key:"Code",op:"NEQ",value:'DataDictionary'}
                             ];
               if(this.getUserInfo().user.roleName!='超级管理员'){
                   inputArr.push( {key:"IsStatic",op:"NEQ",value:1})
               }
                this.mainInput.addqueryConditionItem(inputArr);
                requestGetDataDictionaryPageList(this.mainInput.inputModel).then(data =>{
                            if(data.success){
                                this.tableData=data.result;
                                }
                            else {
                                 this.$message.error('获取数据失败！');
                            }
                          this.mainTableLoading=false;
                }).catch(function(error){
                           _this.mainTableLoading=false;
                });
        },
        _currentChange(val){
                //单选时的方法
                if(val!=null){
                     this.selectRows=[];
                     this.selectRows.push(val);
                }
            },
        toggleRowSelection (row) {
            this.$refs.table.toggleRowSelection(row)
        },
        _reload(){
            this._getDataDictionaryDetailPageList();
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
                console.log(this.selectRows[0]);
                let deleted=true;
                if(this.selectRows[0].isStatic&&this.getUserInfo().user.roleName!=='超级管理员'){
                    deleted=false;
                    this.$message.warning('系统参数没有权限不能操作！')
                }

                if(deleted){
                    requestDeleteDataDictionary(obj).then(data =>{
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
                                        message: '删除数据失败！',
                                        });
                                }
                    });
                }
            });
        },
        _close(bool){
            if(bool){
                this.addFormVisible=false;
            }
            this._reload();
        },
        _treeReload(){
                this._getGetDataDictionaryTreeList();
            }, 
    },
    components:{
         Edit
        },
    mounted(){
        this._reload();
        this._getGetDataDictionaryTreeList();
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus" rel="stylesheet/stylus" scoped>
</style>

