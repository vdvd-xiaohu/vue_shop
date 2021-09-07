<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>分类参数</el-breadcrumb-item>
        </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
        <!-- 提示框区域 -->
          <el-alert
         title="注意：只允许第三级分类设置相关参数"
         type="warning" :closable="false" show-icon>
    </el-alert>
    <!-- 选择商品分类区域 -->
    <el-row class="cat_opt">
        <el-col>
            <span>选择商品分类：</span>
            <!-- 选择商品的级联选择框 -->
         <el-cascader
         expand-trigger="hover"
    v-model="selectedCateKeys"
    :props="cateProps"
    :options="CateList"
    @change="handleChange"></el-cascader>
        </el-col>
    </el-row>

    <!-- tab页标签 -->
     <el-tabs v-model="activeName" @tab-click="handleTabClick">
    <el-tab-pane label="动态参数" name="many">
        <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible=true">添加参数</el-button>
        <!-- 动态参数表格 -->
        <el-table :data="manyTableDate" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand"></el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作" >
                <template slot-scope="scope">
                    <el-button type="primary" size="mini" icon="el-icon-edit">编辑</el-button>
                    <el-button type="danger" size="mini" icon="el-icon-search">删除</el-button>

                </template>
            </el-table-column>
        </el-table>
    </el-tab-pane>
    <el-tab-pane label="静态属性" name="only">
        <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible=true">添加属性</el-button>
<el-table :data="onlyTableDate" border stripe>
            <!-- 展开行 -->
            <el-table-column type="expand"></el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="属性名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作" >
                <template slot-scope="scope">
                    <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditDialog()">编辑</el-button>
                    <el-button type="danger" size="mini" icon="el-icon-search">删除</el-button>

                </template>
            </el-table-column>
        </el-table>

    </el-tab-pane>
    
  </el-tabs>
    </el-card>
    <!-- 添加参数的对话框 -->
   
  
    </div>
</template>

<script>
export default {
    data() {
        return {
            CateList:[],
            // 级联选择框双向绑定到的数组
            selectedCateKeys:[],
           cateProps:{
               value:'cat_id',
               label:'cat_name', 
               children:'children'
           },
           activeName:'many',
        //    动态参数的数据
        manyTableDate:[],
        onlyTableDate:[],
        // 控制添加对话框的显示和隐藏
        addDialogVisible:false,
        // 控制修改对话框的修改与隐藏
        editDialogVisible:false,
        addForm:{},
        // 修改的表单数据对象
        editForm:{},

        // 添加表单的验证规则对象
        addFormRules:{
            attr_name:[{
                required:true,
                message:'请输入参数名称',
                trigger:'blur'
            },]
        },
        // 修改表单的验证规则对象
        editFormRules:{
             attr_name:[{
                required:true,
                message:'请输入参数名称',
                trigger:'blur'
            },]
        }
        }
    },
    methods:{
     async  getCateList() {
        const {data:res} = await this.$http.get('categories')
        //  console.log(res.data)
        if(res.meta.status != 200){
            return this.$message.error('获取分类列表失败')
        }
        this.CateList = res.data
        },
        // 级联选择框选中项变化，会触发这个函数
      async  handleChange() {
            this.getParamsDate()
        },
        handleTabClick() {
            console.log(this.activeName)
            this.getParamsDate()
        },
      async  getParamsDate() {
              if(this.selectedCateKeys.length !== 3) {
                this.selectedCateKeys = []
                return 
            }
            console.log(this.selectedCateKeys)
            //根据所选分类的id，和当前所处的面板，获取对应的参数
      const {data:res} = await this.$http.get('categories/$(this.cateId)/attributes',{
                params:{ sel:this.activeName }
            })
            if(res.meta.status !== 200) {
                return this.$message.error('获取参数列表失败！')
            }
            console.log(res.data)
            if(this.activeName === 'many') {
                this.manyTableDate = res.data
            } else {
                this.onlyTableDate = res.dada
            }
        },
        // 监听添加对话框的关闭事件
        addDialogClosed() {
            this.$refs.addFormRef.resetFields()
        },
        // 点击按钮，添加参数
        addParams() {
           
        },
        showEditDialog() {
            this.editDialogVisible = true
        },
        editDialogClosed() {
            this.$refs.editFormRef.resetFields()
        },
        // 点击按钮，修改参数信息
        editParams() {

        }
    },
    created() {
        this.getCateList()
    },
    computed:{
        isBtnDisabled() {
            if(this.selectedCateKeys !== 3){
                return true
            }
            return  false
        },
        cateId() {
            if(this.selectedCateKeys.length === 3) {
                return this.selectedCateKeys[2]
            }
            return null  
        },
        // 动态计算标题的文本
        titleTeXt() {
            if(this.activeName === 'many') {
                return '动态参数'
            }
            return '静态属性'
        }
    }
}
</script>

<style scoped>
.cat_opt {
    margin-top: 15px;
}
</style>


