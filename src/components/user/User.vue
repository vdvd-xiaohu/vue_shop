<template>

    <div>
        <!-- 面包屑导航区域  -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>


<el-card >
 <!-- 搜索添加区域区域 -->
 

  <el-row :gutter="20">
  <el-col :span="9">
      <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList"> 
    <el-button slot="append" icon="el-icon-search"  @click="getUserList"></el-button>
  </el-input> 
  </el-col>
  <el-col :span="4">
      <el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
  </el-col>
  
</el-row>

<!-- 表格渲染区域 -->
 <el-table
      :data="userList"
      style="width: 100%"
      border stripe
      >
       <el-table-column type="index">
      </el-table-column>
      <el-table-column
        prop="username"
        label="姓名"
        >
      </el-table-column>
      <el-table-column
        prop="email"
        label="邮箱">
      </el-table-column>
      <el-table-column
        prop="mobile"
        label="电话">
      </el-table-column>
       <el-table-column
        prop="role_name"
        label="角色">
      </el-table-column>

        <el-table-column
        prop="mg_state"
        label="状态">
        <template slot-scope="scope">
             <el-switch v-model="scope.row.mg_state" @change="underStateChange(scope.row)"></el-switch>
        </template>
      </el-table-column>

        <el-table-column
        label="操作" width="180px">
         <template slot-scope="scope">
             <!-- 修改按钮 -->
             <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
             <!-- 删除按钮 -->
             <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)"></el-button>
             <!-- 权限按钮 -->
            
             <el-tooltip  effect="dark" content="分配角色" placement="top" >
                <el-button type="warning" icon="el-icon-setting" size="mini" @click="setRole(scope.row)"></el-button>
             </el-tooltip>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页区域 -->
     <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[1, 2, 5, 10]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
</el-card>
<!-- 添加用户的对话框  -->
<el-dialog
  title="添加用户"
  :visible.sync="addDialogVisible"
  width="30%"
  @close="addDialogClosed"
  >
  <!-- 内容主体区域 -->
  <el-form ref="addFormRef" :model="addForm" :rules="addFormRules" label-width="70px">
  <el-form-item label="用户名" prop="username">
    <el-input v-model="addForm.username"></el-input>
  </el-form-item>
   <el-form-item label="密码" prop="password">
    <el-input v-model="addForm.password"></el-input>
  </el-form-item>
  <el-form-item label="邮箱" prop="email">
    <el-input v-model="addForm.email"></el-input>
  </el-form-item>
 
  <el-form-item label="手机" prop="phone">
    <el-input v-model="addForm.phone"></el-input>
  </el-form-item>
  </el-form>
  <!-- 底部区域 -->
  <span slot="footer" class="dialog-footer">
    <el-button @click="addDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addUser">确 定</el-button>
  </span>
</el-dialog>
<!-- 添加修改的对话框 -->
<el-dialog
  title="修改"
  :visible.sync="editDialogVisible"
  width="50%" @close="showEditDialogClosed"
  >
 <el-form  :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
  <el-form-item label="用户名">
    <el-input v-model="editForm.username" disabled></el-input>
  </el-form-item>
   <el-form-item label="邮箱" prop="email">
    <el-input v-model="editForm.email" ></el-input>
  </el-form-item>
   <el-form-item label="手机" prop="mobile">
    <el-input v-model="editForm.mobile" ></el-input>
  </el-form-item>
 </el-form>
  <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editUserInfo">确 定</el-button>
  </span>
</el-dialog>
<!-- 分配角色的对话框 -->
<el-dialog
  title="分配角色"
  :visible.sync="setRoleDialogVisible"
  width="50%"
  >
  <div>
      <p>当前的用户：{{userInfoo.username}}</p>
      <p>当前的角色：{{userInfoo.role_name }}</p>
     <el-select v-model="rolesList" placeholder="请选择">
    <el-option
      v-for="item in rolesList"
      :key="item.id"
      :label="item.roleName"
      :value="item.id"
      >
    </el-option>
  </el-select>
  </div>
  <span slot="footer" class="dialog-footer">
    <el-button @click="setRoleDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
  </span>
</el-dialog>
    </div>
</template>
<script>
export default {
    data() {
        // 验证email的规则
        var checkEmail = (rule,value,cb) => {
             const regEmail = /^\w+@\w+(\.\w+)+$/
             if(regEmail.test(value)){
                 return cb()
             }
             cb(new Error('请输入合法的邮箱'))
        }
        // 验证手机号的规则
        var checkPhone = (rule,value,cb) => {
             const regMobile = /^1[34578]\d{9}$/
    if (regMobile.test(value)) {
      return cb()
    }
    //返回一个错误提示
    cb(new Error('请输入合法的手机号码'))
        }
        return {
            // 获取列表数据的参数
            queryInfo:{
                query:'',
                pagenum:1,
                pagesize:2
            },
            userList:[],
            total:0,
            addDialogVisible:false,
            editDialogVisible:false,
            setRoleDialogVisible:false,
            userInfo:{},
            rolesList:[],
            //查询到的用户信息对象
            editForm:{},
            userInfoo:{},
            addForm:{
                username:'',
                password:'',
                email:'',
                phone:''
            },
            addFormRules:{
                username:[
                    { required:true,message:'请输入用户名',trigger:'blur'},
                    { min:3,max:10,message:'用户名的长度在3到10个字符之间'}
                ],
                password:[
                     { required:true,message:'请输入密码',trigger:'blur'},
                    { min:6,max:15,message:'用户名的长度在6到15个字符之间'}
                ],
                email:[
                       { required:true,message:'请输入邮箱',trigger:'blur'},
                   
                    {validator:checkEmail,trigger:'blur'}
                ],
                phone:[
                     { required:true,message:'请输入手机号',trigger:'blur'},
                   
                     {validator:checkPhone,trigger:'blur'}
                ]
            },
            editFormRules:{
                email:[
                      { required:true,message:'请输入邮箱',trigger:'blur'},
                      {validator:checkEmail,trigger:'blur'}
                ],
                mobile:[
                     { required:true,message:'请输入手机号',trigger:'blur'},
                   
                     {validator:checkPhone,trigger:'blur'}
                ]
            },
           

        }
    },
    methods:{
      async  getUserList() {
           const {data:res} =await this.$http.get('users',{params:this.queryInfo})
           if(res.meta.status !== 200) return this.$message.error('获取用户列表失败！')
            this.userList = res.data.users
            this.total = res.data.total
           //console.log(res)
        },
         handleSizeChange(newsize) {
        this.queryInfo.pagesize = newsize
        this.getUserList()

    },
    handleCurrentChange(newsize) {
        this.queryInfo.pagenum = newsize
        this.getUserList()

    },
    async underStateChange(userinfo) {
        console.log(userinfo)
      const {data:res} = await this.$http.put('users/${userinfo.id}/state/${userinfo.mg_state}')
      if(res.meta.status !== 200) {
          userinfo.mg_state = !userinfo.mg_state
          return this.$message.error('更新用户状态失败')
          
      }
      this.$message.success('更新用户状态成功')

    },
     addDialogClosed() {
        this.$refs.addFormRef.resetFields()
    },
    addUser() {
        this.$refs.addFormRef.validate(async valid =>{
            if(!valid) return 
         const {data:res} = await this.$http.post('users',this.addForm)
         if(res.meta.status!==201) {
             this.$message.error('添加用户失败')
         }
        this.$message.success('添加用户成功')
        // 隐藏添加的对话框
        this.addDialogVisible = false
        // 重新获取用户的列表
        this.getUserList()
        })
    },
   async showEditDialog(id) {
       // console.log(id)
      const {data:res} = await this.$http.get('users/'+id)
      if(res.meta.status !== 200) {
          return this.$message.error('查询用户信息失败')
      }
    this.editForm = res.data
        this.editDialogVisible = true
    },
    showEditDialogClosed() {
        this.$refs.editFormRef.resetFields()
    },
    // 修改用户信息并提交
    editUserInfo() {
        this.$refs.editFormRef.validate(async valid => {
            if(!valid) return
            //发起数据请求
        const {data:res} =  await this.$http.put('users/'+this.editForm.id,{
                email:this.editForm.email,
                mobile:this.editForm.mobile
            }) 
    if(res.meta.status !== 200) {
        return this.$message.error('更新用户信息失败')
    }
    this.editDialogVisible = false
    this.getUserList()
    this.$message.success('更新数据成功')
        })
    },
  async   removeUserById(id) {
        console.log(id)
       const res = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch((err)=>{
            return err
        })
        if(res !== 'confirm') {
            return this.$message.info('您已经取消了删除')
        }
      const {data:ress} = await this.$http.delete('users/'+id)
      if(ress.meta.status !== 200) {
          return this.$message.error('用户删除失败了')
      }
       this.$message.success('用户删除成功')
        this.getUserList()
       
    },
   async  setRole(userInfo) {
         console.log(userInfo)
         this.userInfoo=userInfo
    const {data:res} = await this.$http.get('roles')
    if(res.meta.status !==200) {
        return this.$message.error('请求角色列表失败')
    }
    this.rolesList = res.data
    this.setRoleDialogVisible = true
            },
         async   saveRoleInfo() {
                if(!this.selectedRoleId) {
                    return this.$message.error('请选择要分配的角色')
                }
             const {data:res} = await this.$http.put('users/${this.userInfo.id}/role',{
                    rid:this.selectedRoleId
                })
                if(res.meta.status !== 200) {
                    return this.$message.error('更新角色失败')
                }
                this.$message.success('更新角色成功')
                this.getUserList()
                this.setRoleDialogVisible = false
            }
    },
    created()  {
        this.getUserList()
    },
   
   
}
</script>
<style scoped>

</style>
