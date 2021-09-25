<template>
     <div>
        <!-- 面包屑导航区 -->
       <el-breadcrumb separator-class="el-icon-arrow-right">
           <el-breadcrumb-item :to= "{path: '/home'}">首页</el-breadcrumb-item>
           <el-breadcrumb-item>商品管理</el-breadcrumb-item>
           <el-breadcrumb-item>商品列表</el-breadcrumb-item>
       </el-breadcrumb>
       <!-- 卡片视图区域 -->
        <el-card>
           <!-- 搜索与添加区域 -->
           <el-row :gutter="20">
               <!-- gutter表示列之间的间隙  span里面的数字代表搜索框的宽度 -->
               <el-col :span="8">
                   <!-- clearable clear属性实现清空搜索框功能 -->
                   <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable  @clear="getGoodsList">
                      <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
                   </el-input>
               </el-col>
               <el-col :span="4">
                   <el-button type="primary" @click="goAddpage">添加商品</el-button>
               </el-col>
           </el-row>
           <!-- table表格区域 -->
           <el-table :data="goodsList" border stripe >
               <el-table-column type="index"></el-table-column>
               <el-table-column  prop="goods_name" label="商品名称"></el-table-column>
               <el-table-column  prop="goods_price" label="商品价格（元）" width="95px"></el-table-column>
               <el-table-column  prop="goods_weight" label="商品重量" width="70px"></el-table-column>
               <el-table-column  prop="add_time" label="创建时间" width="140px">
                   <template slot-scope="scope">
                       <!-- dataFormat为在main.js中定义的全局时间转换函数 -->
                           {{scope.row.add_time | dateFormat}}
                   </template>
               </el-table-column>
               <el-table-column  label="操作" width="130px">
                   <template slot-scope="scope">
                        <!-- 修改商品按钮 -->
                       <el-button type="primary" icon="el-icon-edit" size="mini"></el-button>
                       <!-- 删除商品按钮 -->
                       <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeById(scope.row.goods_id)"></el-button>
                   </template>
               </el-table-column>
           </el-table>
           <!-- 分页区域 -->
            <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
           :current-page="queryInfo.pagenum" :page-sizes="[5,10,15,20]" :page-size="queryInfo.pagesize"
           layout="total, sizes, prev, pager, next, jumper"  :total="total" background >
           </el-pagination>
        </el-card>
    </div>
</template>

<script>
export default {
  data() {
    return {
      // 查询参数对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      // 商品列表
      goodsList: [],
      // 商品总数据条数
      total: 0
    }
  },
  created() {
    this.getGoodsList()
  },
  methods: {
    // 根据分页获取对应的商品列表
    async getGoodsList() {
      const { data: res } = await this.$http.get('goods', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表失败！')
      }
      this.$message.success('获取商品列表成功! ')
      console.log(res.data)
      this.goodsList = res.data.goods
      this.total = res.data.total
    },
    // 获取页码
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsList()
    },
    // 当前页码值
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getGoodsList()
    },
    async removeById(id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该参数，是否继续？',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // 用户取消了删除操作
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      // 确认删除
      const { data: res } = await this.$http.delete(`goods/${id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败！ ')
      }
      this.$message.error('删除成功！ ')
      this.getGoodsList()
    },
    goAddpage() {
      this.$router.push('goods/add')
    }

  }

}
</script>

<style lang="less" scoped>
</style>
