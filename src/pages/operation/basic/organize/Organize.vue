﻿<template>
<yl-panelpage :titleName="'组织机构管理'" >
<div slot="content">
  <yl-columnlay>
               <div slot="left" >        
                   <yl-organizeTreeWithPanel :highlightCurrent="true"  :treeLoading="treeLoading" :reloadIsShow="true" v-model="origanizeWithPanel" @getCurrentNode="_getCrureentNode"  :placeholder="'请选择'"   style="width:200px"> </yl-organizeTreeWithPanel>
                </div>
                <div slot="right" >
                    <yl-layout>
                        <div slot="fristbox">
                          <yl-toolbar>
                                <el-form  :inline="true"  >
                                        <el-form-item class="form-content-vertical">
                                       <el-input  placeholder="部门简称" size="small" style="width:200px" v-model="searchModel.shortName"></el-input>  
                                        </el-form-item>
                                        <el-form-item class="form-content-vertical">
                                            <el-button type="primary" icon="el-icon-search" size="small"  @click="_reload">查 询</el-button>
                                        </el-form-item>
                                </el-form>
                          </yl-toolbar>
                        <yl-toolbar>
                            <el-button-group>
                                    <el-button type="primary" icon="el-icon-plus" size="small" v-permissionSetting="'Organizes.Create'" @click="_add" >添加</el-button>
                                    <el-button type="primary" icon="el-icon-edit"  size="small" v-permissionSetting="'Organizes.Edit'" :disabled="selectRows.length===0" @click="_edit">编辑</el-button>
                                    <el-button type="danger"  icon="el-icon-delete" size="small"  v-permissionSetting="'Organizes.Delete'" :disabled="selectRows.length===0" @click="_delete" >删除</el-button>
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
                                </yl-table>
                         </div>
                           </yl-layout>
                     </div>
     </yl-columnlay>
    <!--编辑界面-->
         <el-dialog ref="mainDialog" 
         title="编辑" 
         :visible.sync="addFormVisible"
         width="55%"
         top="5%" 
                  :modal-append-to-body="false" :close-on-click-modal="false"  >
                    <Edit :selectRow="currentRows" @close="_close"  v-if="addFormVisible"></Edit>
         </el-dialog>
 </div>
</yl-panelpage>
</template>
<script type="text/babel">
import Edit from './OrganizeEdit';
import util from '@/common/js/util';
import  {inputModel} from '@/api/inputmodel';
import organizeTreeWithPanel from '@/ocomponents/organize/organizeTreeWithPanel'
import {
         requestGetOrganizePageList,
         requestDeleteOrganize
        }from '@/api/organize';
export default {
    data(){
        return {
             tableData: {},
                mainTableLoading:false,
                selectRows:[],
                currentRows:{},              
                origanizeWithPanel:[],
                searchModel:{
                shortName:'',  
                    },  
                addFormVisible:false,
                mainInput:new inputModel(),
                selectNode:{id:"",code:''},
                defaultProps: {
                        children: 'children',
                        label: 'text',
                        id: 'id'
                    },
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
                                    {attr: { type: 'index',label: '序号', width:55,align: 'center',headerAlign:'center' }},
                                    {attr: {prop:"nodeCode",label:"组织编码",width:"150",  } },
                                    {attr: {prop:"category",label:"部门类别",width:"120",  } },
                                    {attr: { prop:"shortName", label:"部门简称", width:"140",  } },
                                    {attr: {prop:"abbName",label:"部门全称",width:"200",  } },
                                    {attr: {prop:"id",label:"组织唯一编码",width:"250",  } },
                                    {attr: {prop:"outerPhone",label:"电话",width:"160",  } },
                                    {attr: {prop:"fax",label:"传真",width:"160",  } },
                                    {attr: {prop:"web",label:"网址",width:"160",  } },
                                    {attr: {prop:"manager",label:"主负责人",width:"120",  } },
                                    {attr: {prop:"address",label:"详细地址",width:"180",  } },
                                   // {attr: {prop:"orgDescription",label:"描述信息",width:"200",  } },
                                    {attr: { prop: 'sortCode', label: '排序', width:70,align: 'center',headerAlign:'center' } },
                            ]
                        }
                    }
                }
        },
    methods:{
        _getOrganizePageList(){    
                var _this=this;
                this.selectRows=[];
                this.mainTableLoading=true; //开启加载
                var inputArr=[ 
                             {key:"ShortName",op:"LIKE",value:this.searchModel.shortName},
                             {key:"IsStatic",op:"EQ",value:false},
                             {key:"NodeCode",op:"LIKEL",value:this.selectNode.code,relation:"or"},
                             {key:"ZhbId",op:"EQ",value:this.selectNode.id==''?'CDDFC606-C3BA-4DD2-907A-481C595DC6BD':this.selectNode.id}
                             ];
                this.mainInput.inputModel.sorting="NodeCode";
        this.mainInput.addqueryConditionItem(inputArr);
         requestGetOrganizePageList(this.mainInput.inputModel).then(data =>{
                            if(data.success){
                                this.tableData=data.result;
                                }
                            else {
                                 this.$message.error('失败！'+data.error.message);
                            }
                          this.mainTableLoading=false;
                }).catch(function(error){
                       _this.mainTableLoading=false;
                });
        },
                 //点击加载 
     
   _getCrureentNode(node){
            this.origanizeWithPanel=node.text;
            this.selectNode.id=node.id;
            this.selectNode.code=node.code;
            this._reload(); 
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
            this._getOrganizePageList();
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
              requestGetOrganizePageList(this.mainInput.inputModel).then(data =>{
               if(data.result.data.length>0){
                     this.$message.error('删除失败！该组织下还有未删除的子节点！');
                     this._reload();
               }else{
                   this.$confirm('确认要删除所选项目吗?', '提示', {
                type: 'warning'
            }).then(() => {
                var obj={}; 
                obj.id=this.selectRows[0].id;
                requestDeleteOrganize(obj).then(data =>{
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
        _close(bool){
            if(bool){
                this.addFormVisible=false;
            }
            this._reload();
        },
    },
    components:{
         Edit,
        'yl-organizeTreeWithPanel':organizeTreeWithPanel,
        },
    mounted(){
        this.selectNode.code=this.getUserInfo().user.orgCode;
        this._reload();
    }
}
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus" rel="stylesheet/stylus" scoped>
</style>

