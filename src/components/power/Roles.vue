<template>
    <div>
        <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>权限管理</el-breadcrumb-item>
        <el-breadcrumb-item>角色列表</el-breadcrumb-item>
</el-breadcrumb>

<!-- 卡片视图 -->
<el-card>
    <!-- 添加按钮区域 -->
    <el-row>
        <el-col>
            <el-button type="primary">添加角色</el-button>
        </el-col>
    </el-row>
    <!-- 角色列表区域 -->
    <el-table :data="rolelist" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
            <template slot-scope="scope">
                <el-row :class="['bdbottom',i1 === 0? 'bdtop':'']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
                    <!-- 渲染一级权限 -->
                    <el-col :span="5">
                      <el-tag closable  @close="removeRightsByid(scope.row,item1.id)">
                            {{ item1.authName }}
                      </el-tag>
                      <i class="el-icon-caret-right"></i>
                    </el-col>
                    <!-- 渲染二级和三级权限 -->
                    <el-col :span="19">
                        <el-row :class="[i2 == 0? '':'bdtop']" v-for="(item2,i2) in item1.children" :key="item2.id">
                            <el-col :span="6">
                               <el-tag closable @close="removeRightsByid(scope.row,item2.id)" type="success">{{ item2.authName }}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <el-col :span="18" v-for="(item3,i3) in item2.children" :key="item3.id">
                                <el-tag closable type="warning" @close="removeRightsByid(scope.row,item3.id)"> {{ item3.authName }} </el-tag>
                            </el-col>
                        </el-row>
                    </el-col>
                </el-row>
              
            </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleName"></el-table-column>
        <el-table-column label="操作" width="300px">
            <template slot-scope="scope">
                <el-button size="mini" type="primary" icon="el-icon-search">编辑</el-button>
                <el-button size="mini" type="danger" icon="el-icon-search">删除</el-button>
                <el-button size="mini" type="warning" icon="el-icon-search" @click="showSetRightsDialog(scope.row)">分配权限</el-button>
            </template>
        </el-table-column>
      
    </el-table>
</el-card>
<!-- 分配权限的对话框 -->

<el-dialog
  title="分配权限"
  :visible.sync="setRightDialogVisible"
  width="50%"
  @close = "setRightDialogClosed"
  >
  <el-tree :data="rightList" :props="defaultProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref
  ="treeRef"></el-tree>
  <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="allotRights">确 定</el-button>
  </span>
</el-dialog>

    </div>
</template>
<script>
export default {
    data() {
        return {
            rolelist:[],
            // 确定分配权限对话框的显示与隐藏
            setRightDialogVisible:false,
            rightList:[],
            defaultProps:{
                children:'children',
                label:'authName'
            },
            defKeys:[],
            roleId:''
        }
    },
    created() {
        this.getRolesList()
    },
    methods:{
        // 获取所有角色的列表
    async    getRolesList() {
       const {data:res} = await  this.$http.get('roles')
       if(res.meta.status !== 200) {
           return this.$message.error('请求角色列表失败')
       }
       this.rolelist = res.data
       console.log(this.rolelist)
        },
    async  removeRightsByid(role,rightId) {
            //弹框提示是否需要删除
        const res =  await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err=>err)
        if(res!=='confirm') {
            this.$message.info('取消了删除')
        }
       const {data:ress} = await this.$http.delete('roles/${role.id}/rights/${rightId}')
        //console.log(ress)
             if(ress.meta.status !== 200) {
                 this.$message.info('删除失败')
             }
            // this.getRolesList()
            role.children = ress.data
        },
        // 展示分配权限的对话框
    async    showSetRightsDialog(role) {

            // 获取所有权限列表数据
            this.roleId = role.id
        const {data:res} = await  this.$http.get('rights/tree')
        if(res.meta.status!==200) {
            return this.$message.error('获取权限列表数据失败')
        }
        this.rightList = res.data
        this.getLeafkeys(role,this.defKeys)
        this.setRightDialogVisible = true
        },
        getLeafkeys(node,arr) {
            if(!node.children) {
                return arr.push(node.id)
            }
            node.children.forEach(item => this.getLeafkeys(item,arr))
        },
        setRightDialogClosed() {
            this.defKeys = []
        },
      async  allotRights() {
            const keys = [
                ...this.$refs.treeRef.getCheckedKeys(),
                ...this.$refs.treeRef.getHalfCheckedKeys(),
            ]
            const idStr = keys.join(',')
    const {data:res} = await   this.$http.post('roles/${this.roleId}/rights',{rids:idStr})
    //console.log(res)
    if(res.meta.status !==200){
        return this.$message.error('更新权限失败！')
    }
    this.$message.success('分配权限成功！')
    this.getRolesList()
    this.setRightDialogVisible = false
        }
    }
}
</script>
<style scoped>
.el-tag {
    margin: 7px;
}
.bdtop {
    border-top: 1px solid #eee;
}
.bdbottom {
    border-bottom: 1px solid #eee;
}
</style>


