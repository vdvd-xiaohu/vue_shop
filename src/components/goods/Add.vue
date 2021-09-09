<template>

    <div>
         <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>添加商品</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区域 -->
        <el-card>
            <!-- 提示区域 -->
    <el-alert
    title="添加商品信息"
    type="info"
    center
    show-icon :closable="false">
  </el-alert>
    <!-- 步骤条 -->
    <el-steps :space="200" :active="activeIndex-0" finish-status="success" align-center>
    <el-step title="基本信息"></el-step>
    <el-step title="商品参数"></el-step>
    <el-step title="商品属性"></el-step>
    <el-step title="商品图片"></el-step>
    <el-step title="商品内容"></el-step>
    <el-step title="完成"></el-step>
    </el-steps>
    <!-- tab栏区域 -->
    <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position="top">
  <el-tabs :tab-position="'left'" v-model="activeIndex" :before-leave = "beforeTabLeave" @tab-click="tabClicked">
    <el-tab-pane label="基本信息" name="0">
      
  <el-form-item label="商品名称" prop="goods_name">
    <el-input v-model="addForm.goods_name"></el-input>
  </el-form-item>

    <el-form-item label="商品价格" prop="goods_price">
    <el-input v-model="addForm.goods_price" type="number"></el-input>
  </el-form-item>

   <el-form-item label="商品重量" prop="goods_weight">
    <el-input v-model="addForm.goods_weight" type="number"></el-input>
  </el-form-item>

     <el-form-item label="商品数量" prop="goods_number">
    <el-input v-model="addForm.goods_number" type="number"></el-input>
  </el-form-item>

    <el-form-item label="商品分类" prop="goods_cat">

         <el-cascader
    v-model="addForm.goods_cat"
    :options="cateList"
    :props="cateProps"
   
    @change="handleChange"></el-cascader>

    </el-form-item>
    </el-tab-pane>
    <el-tab-pane label="商品参数" name="1">
        <!-- 渲染参数的ITEM项 -->
        <el-form-item :label="item.attr_name" v-for="item in manyTableData" :key="item.attr_id"></el-form-item>

    </el-tab-pane>
    <el-tab-pane label="商品属性" name="2">商品属性</el-tab-pane>
    <el-tab-pane label="商品图片" name="3">商品图片</el-tab-pane>
    <el-tab-pane label="商品内容" name="4">商品内容</el-tab-pane>
  </el-tabs>
    </el-form>

        </el-card>
    </div>
</template>
<script>
export default {
    data() {
        return {
            activeIndex:'0',
          manyTableData:[],
           

            // 添加表单的数据对象
           addForm:{
               goods_price:0,
               goods_name:'',
               goods_weight:0,
               goods_number:0,
            //    商品所属的分类数组
               goods_cat:[] 
           },
           cateList:[],
           cateProps:{
               label:'cat_name',
               value:'cat_id',
               children:'children'
           },
           addFormRules:{
               goods_name:[
                   {required:true,message:'请输入商品名称',trigger:'blur'},
               ],
               goods_price:[
                    {required:true,message:'请输入商品价格',trigger:'blur'},
               ],
                goods_weight:[
                    {required:true,message:'请输入商品重量',trigger:'blur'},
               ],
                goods_number:[
                    {required:true,message:'请输入商品数量',trigger:'blur'},

                ],
                goods_cat:[
                    {required:true,message:'请选择商品分类',trigger:'blur'}
                ]
           },


        }
    },
    methods:{
       async getCateList() {
         const {data:res} = await this.$http.get('categories')

         if(res.meta.status !== 200) {
             return this.$message.error('获取商品分类列表失败！')
         }
         this.cateList = res.data
         console.log(this.cateList)
        },
        // 级联选择器选中项发生变化，就会触发此函数
        handleChange() {
            console.log(this.addForm.goods_cat)
            if(this.addForm.goods_cat.length !== 3) {
                this.addForm.goods_cat = []
            }
        },
        beforeTabLeave(activeName,oldActiveName) {
                // console.log('即将离开的标签页名字是'+oldActiveName)
                // console.log('即将进入的标签页名字是'+activeName)
                if(oldActiveName==='0'&& this.addForm.goods_cat.length!== 3){
                    this.$message.error('请先选择商品分类')
                    return false
                }
        },
    async   tabClicked() {
            if(this.activeIndex === '1') {
                console.log('动态参数面板')
         const {data:res}  = await  this.$http.get('categories/${this.cateId}/attributes',{
                    params:{sel:'many'}
                })
                if(res.meta.status!== 200) {
                    return this.$message.error('获取列表数据失败')
                }
                console.log(res.data)
                res.data.forEach(item => {
                  item.attr_vals = item.attr_vals.length === 0 ?
                    []: item.attr_vals.split(' ')
                })
               
            }
        }
    },
    computed:{
    cateId() {
        if(this.addForm.goods_cat.length === 3) {
            return this.addForm.goods_cat[2]
        }
    }
    },
    created() {
        this.getCateList()
    }
}
</script>

<style scoped>

</style>


