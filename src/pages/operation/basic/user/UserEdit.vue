<template>
<div>
  <el-form  :model="formModel" status-icon  :rules="rules" ref="formModel" label-width="100px" >
   <el-tabs v-model="activeName" @tab-click="_handleNodeClick" class="el-tab" >
    <el-tab-pane label="用户管理" name="first" style="height:250px">
       <el-row>
            <el-form-item label="所在单位" prop="orgId">
            <yl-organizeTree  
                v-model="formModel.orgId" 
                 @getCurrentNode="_getCrureentNode" 
                :placeholder="'请选择所在单位'"  
                :isexpand="false">
            </yl-organizeTree>
            </el-form-item> 
        </el-row>
        <el-row>
          <el-col :span="12">    
            <el-form-item label="姓名" prop="realName">
                <el-input v-model="formModel.realName" ></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="12">
                <el-form-item label="用户名" prop="userName">
                    <el-input v-model="formModel.userName" >
                    </el-input>
                </el-form-item>
           </el-col>
        </el-row>
    
         <el-row>   
                <el-form-item label="电话号码" prop="telephoneNo"> 
                        <el-input v-model="formModel.telephoneNo" ></el-input>
                </el-form-item>
        </el-row> 
         <el-row> 
                <el-form-item label="电子邮件" prop="emailAddress"> 
                  <el-input v-model="formModel.emailAddress" ></el-input>
                </el-form-item>
        </el-row>
        <el-row>
      <el-col :span="12">
        <el-form-item label="性别" prop="gender"> 
            <el-radio-group v-model="formModel.gender"  size="small">
                <el-radio label="男">男</el-radio>
                <el-radio label="女">女</el-radio>
            </el-radio-group>
        </el-form-item>
        </el-col>
        <el-col :span="12">
                <el-form-item label="状态" prop="isActive">
                    <el-switch  v-model="formModel.isActive"
                                    active-text="启用"
                                    inactive-text="禁用"
                                    active-color="#13ce66"
                                    inactive-color="#ff4949">
                        </el-switch>
                 </el-form-item>
        </el-col>
        </el-row> 
    </el-tab-pane>
   <el-tab-pane label="用户角色权限" name="second" class="yl-tab-pane">
        <el-tree
        :data="roleTreeData"
        :check-strictly="true"
        :empty-text="'数据获取中...'"
        show-checkbox
        default-expand-all
        node-key="id"
        ref="roleTree"
        highlight-current
        :props="defaultProps">
        </el-tree>
   </el-tab-pane>
    <el-tab-pane label="用户数据权限" name="three" class="yl-tab-pane">
        <el-tree 
            :data="organizeTreeData"
            show-checkbox
            :empty-text="'数据获取中...'"
            :check-strictly="true"
            node-key="id"
            ref="dataRangeTree"
            highlight-current
            :props="defaultProps">
            </el-tree>
    </el-tab-pane>  
    </el-tabs>  
   <el-form-item style="text-align:right;" >
      <el-button icon="el-icon-refresh" @click="_resetForm">重置</el-button>
      <el-button icon="el-icon-circle-check-outline" type="primary" @click="_onSubmit" :loading="loading">保存</el-button>
  </el-form-item>
</el-form>
</div>
</template>
<script type="text/babel">
import {
    requestCreateOrUpdateUser,
    requestGetOrganizeTreeList,
    requestGetRoleTreeList,
    requestGetUserForEdit
}from '@/api/user';
import util from '@/common/js/util';
import organizeTree from '@/ocomponents/organize/organizeTree'
export default {
  data(){
     
    return{
        formModel:{
                 id:'',
                 orgId:'',
                 orgCode:'',
                 orgFullPathName:'',
                 realName:'',
                 spell:'',
                 userName:'',
                 ouserName:'',
                 password:'',
                 gender:'男',
                 telephoneNo:'',
                 emailAddress:'',
                 isEmailConfirmed:'',
                 emailConfirmationCode:'',
                 passwordResetCode:'',
                 lastLoginTime:'',
                 isActive: true,
                 sortCode: 1,
                 userType: 0,
        },
         rules: {
                    realName: [
                      { required: true, message: '姓名不能为空！', trigger: 'blur' }
                    ],
                    userName: [
                      { required: true, message: '用户名不能为空！', trigger: 'blur'},
                      { validator:this.commonJudgeRepeat, trigger: 'blur' }
                    ],
                    orgId: [
                      { required: true, message: '所在单位不能为空！', trigger: 'change' }
                    ], 
                     
       },
        loading:false,
        activeName: 'first', 
        organizeTreeData:[],
        roleTreeData:[],
        selectNode:{id:""},
        defaultProps: {
                children: 'children',
                label: 'text',
                id: 'id'
            },
        orgExpanded:[]
       }       
    },
    props:{
        selectRow:{},
        isVisible:false
    },
    methods:{
         _onSubmit() {
                this.$refs.formModel.validate((valid) => {
                if (valid) {
                    this.loading=true;
                    let params={};
                    params.grantedRoles=this.$refs.roleTree.getCheckedKeys();
                    params.grantedDataRanges=this.$refs.dataRangeTree.getCheckedKeys();
                    params.user=this.formModel;
                    if(params.grantedRoles.length==0||params.grantedDataRanges.length==0){
                        this.$message.error('请给用户赋予角色和权限范围！');
                        this.loading=false;
                        return;
                    }
                    requestCreateOrUpdateUser(params).then(data=>{
                    if(data.success){
                            this.$notify({
                                title: '成功',
                                message: '保存数据成功！',
                                type: 'success'
                                });
                            //关闭面板
                            this._complete();
                        }
                        else {
                            this.$message.error('保存数据失败！');
                        }
                        this.loading=false;
                    }).catch(function(error){
                    this.loading=false;
                  })
                } else {
                    console.log('error submit!!');
                    return false;
                }
            });
            },
         _resetForm() {
                this.$refs.formModel.resetFields();
                this.$refs.roleTree.setCheckedKeys([]);
                this.$refs.dataRangeTree.setCheckedKeys([]);
         },
         _complete(){
                this.$emit('close');
            },
         _handleNodeClick(result, resolve) {  
                    //点击加载 
                    this.selectNode=result;
                }, 
        _getRoleTreeList(){
                var _this=this;
                requestGetRoleTreeList().then(data => {
                    if(data.success){
                       this.roleTreeData=data.result;
                    }
                else {
                    this.$message.error('获取数据失败！'+data.error.message);
                }
                })
            },     
         _getOrganizeTreeList(){
                var _this=this;
                requestGetOrganizeTreeList().then(data => {
                    if(data.success){
                       this.organizeTreeData=data.result;
                    }
                else {
                    this.$message.error('获取数据失败！'+data.error.message);
                }
                })
            },      
         _getUserForEdit(){
                var _this=this;
                let params={};
                params.id=this.formModel.id;
                requestGetUserForEdit(params).then(data => {
                    if(data.success){
                        this.$refs.roleTree.setCheckedKeys(data.result.grantedRoles);
                        this.$refs.dataRangeTree.setCheckedKeys(data.result.grantedDataRanges);
                    }
                    else{
                        this.$message.error('获取数据失败！');
                    }
            })
        },    
        _getCrureentNode(node){
            this.formModel.orgCode=node.code;
        },
        commonJudgeRepeat(rule, value, callback){
            let columnName='userName',columnValue='';
            let columnNameArr=columnName.split(',');
            columnNameArr.map(data=>{
                    columnValue=columnValue+this.formModel[data]+',';
                })
            columnValue=columnValue.substring(0,columnValue.length-1)
            let str=`Authority_User|${columnName}|${columnValue}|${this.formModel.ouserName}`;
            this.IsJudgeRepeat(str,value, callback);
        } 
    },
  mounted(){
         Object.assign(this.formModel,this.selectRow); 
         this._getRoleTreeList();  
         this._getOrganizeTreeList(); 
         this._getUserForEdit();
         this.formModel.ouserName=this.formModel.userName;
    },    
    components:{
       'yl-organizeTree':organizeTree,
    },
    watch:{
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.yl-tab-pane {
    height: 250px;
}
.yl-tab-pane > div{
    height: 100%;
}
</style>
