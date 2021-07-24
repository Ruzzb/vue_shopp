<template>
    <div>
       <!-- 面包屑导航区 -->
       <el-breadcrumb separator-class="el-icon-arrow-right">
           <el-breadcrumb-item :to= "{path: '/home'}">首页</el-breadcrumb-item>
           <el-breadcrumb-item>用户管理</el-breadcrumb-item>
           <el-breadcrumb-item>用户列表</el-breadcrumb-item>
       </el-breadcrumb>
       <!-- 卡片视图区域 -->
       <el-card>
           <!-- 搜索与添加区域 -->
           <el-row :gutter="20">
               <!-- gutter表示列之间的间隙  span里面的数字代表搜索框的宽度 -->
               <el-col :span="7">
                   <!-- clearable clear属性实现清空搜索框功能 -->
                   <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
                      <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
                   </el-input>
               </el-col>
               <el-col :span="4">
                   <el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
               </el-col>
           </el-row>
           <!-- 用户列表区域 -->
           <el-table :data="userlist" border stripe>
               <!-- 索引列 -->
               <el-table-column type="index"></el-table-column>
               <!-- label为列的名称  prop指向要显示的数据 -->
               <el-table-column label="姓名" prop="username"></el-table-column>
               <el-table-column label="邮箱" prop="email"></el-table-column>
               <el-table-column label="电话" prop="mobile"></el-table-column>
               <el-table-column label="角色" prop="role_name"></el-table-column>
               <el-table-column label="状态">
                   <!-- 有了template作用域插槽后就不需要prop="mg_state"，因为后者会将前者覆盖 -->
                   <template slot-scope="scope">
                       <!-- scope.row可以获取到该行对应的数据  switch按钮开关-->
                       <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)"></el-switch>
                   </template>
               </el-table-column>
               <el-table-column label="操作" width="180px">
                   <template slot-scope="scope">
                       <!-- 修改按钮 -->
                       <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
                       <!-- 删除按钮 -->
                       <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)"></el-button>
                       <!-- 分配按钮     tooltip显示提示   :enterable属性控制鼠标是否可以进入图标 -->
                       <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="flase">
                           <el-button type="warning" icon="el-icon-setting" size="mini" @click="setRole(scope.row)"></el-button>
                       </el-tooltip>
                   </template>
               </el-table-column>
           </el-table>
           <!-- 分页区域 -->
           <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
           :current-page="queryInfo.pagenum" :page-sizes="[1,2,5,10]" :page-size="queryInfo.pagesize"
           layout="total, sizes, prev, pager, next, jumper"  :total="queryInfo.total">
           </el-pagination>
       </el-card>
       <!-- 添加用户对话框 -->
       <el-dialog title="添加用户" ：visible.sync="addDialogVisible" width:50%  @close="addDialogClosed">
           <!-- 内容主体区 -->
           <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
               <el-form-item label="用户名" prop="username">
                   <el-input v-model="addForm.username"></el-input>
               </el-form-item>
               <el-form-item label="密码" prop="password">
                   <el-input v-model="addForm.password"></el-input>
               </el-form-item>
               <el-form-item label="邮箱" prop="email">
                   <el-input v-model="addForm.email"></el-input>
               </el-form-item>
               <el-form-item label="手机" prop="mobile">
                   <el-input v-model="addForm.mobile"></el-input>
               </el-form-item>
           </el-form>
           <!-- 底部区域 -->
           <span slot="footer" class="dialog-footer">
           <el-button @click="addDialogVisible=false">取消</el-button>
           <el-button type="primary" @click="addUser">确定</el-button>
           </span>
       </el-dialog>
       <!-- 修改用户的对话框 -->
       <el-dialog title="修改用户" ：visible.sync="editDialogVisible" width:50%  @close="editDialogClosed">
           <!-- 内容主体区 -->
           <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
               <el-form-item label="用户名">
                   <el-input v-model="editForm.username" disabled></el-input>
               </el-form-item>
                <el-form-item label="邮箱" prop="email">
                   <el-input v-model="editForm.email"></el-input>
               </el-form-item>
               <el-form-item label="手机" prop="mobile">
                   <el-input v-model="editForm.mobile"></el-input>
               </el-form-item>
           </el-form>
           <!-- 底部区域 -->
           <span slot="footer" class="dialog-footer">
           <el-button @click="editDialogVisible=false">取消</el-button>
           <el-button type="primary" @click="eitdUserInfo">确定</el-button>
           </span>
       </el-dialog>
       <!-- 分配角色的对话框 -->
        <el-dialog title="分配角色" ：visible.sync="setRoleDialogVisible" width:50% @close="setRoleDialogClosed">
           <div>
             <p>当前的用户:{{userInfo.username}}</p>
             <p>当前的角色:{{userInfo.role_name}}</p>
             <p>分配角色:
               <el-select v-model="selectedRoleId" placeholder="请选择">
                 <el-option v-for="item in rolesList" :key="item.id" :label="item.roleName" :value="item.id"></el-option>
               </el-select>
               </p>
           </div>
           <span slot="footer" class="dialog-footer">
           <el-button @click="setRoleDialogVisible=false">取消</el-button>
           <el-button type="primary" @click="saveRoleInfo">确定</el-button>
           </span>
       </el-dialog>
    </div>
</template>

<script>

export default {
  data() {
    // 验证邮箱的规则
    var checkEmail = (rule, value, cb) => {
      // 验证邮箱的正则表达式
      const regEmail = /^[A-Za-z\d]+([-_.][A-Za-z\d]+)*@([A-Za-z\d]+[-.])+[A-Za-z\d]{2,4}$/
      if (regEmail.test(value)) {
        // 合法邮箱
        return cb
      }
      cb(new Error('请输入合法的邮箱'))
    }
    // 验证手机号的规则
    var checkMobile = (rule, value, cb) => {
      // 验证手机号的正则表达式
      const regMobile = /^[1][3,4,5,7,8][0-9]{9}$/
      if (regMobile.test(value)) {
        // 合法手机号
        return cb
      }
      cb(new Error('请输入合法的手机号'))
    }
    return {
      // 获取用户参数对象
      queryInfo: {
        query: '',
        // 当前页数
        pagenum: 1,
        // 当前每页显示多少条数据
        pagesize: 2
      },
      userlist: [],
      total: 0,
      // 控制添加用户对话框的显示与隐藏
      addDialogVisible: false,
      // 添加用户的表单数据
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 添加表单的验证规则对象
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '用户名的长度在3-10个字符之间', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '密码的长度在6-15个字符之间', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trriger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trriger: 'blur' }
        ]
      },
      // 控制修改用户对话框的显示与隐藏
      editDialogVisible: false,
      // 查询到的用户信息对象
      editForm: {},
      // 修改表单的验证规则对象
      editFormRules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trriger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trriger: 'blur' }
        ]
      },
      // 控制分配角色对话框的显示与隐藏
      setRoleDialogVisible: false,
      // 需要被分配角色的用户信息
      userInfo: {},
      // 所有角色的数据列表
      rolesList: [],
      // 已选中的角色id值
      selectedRoleId: ''
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    async getUserList() {
      const { data: res } = await this.$http.get('users', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取用户列表失败！')
      }
      // 获取用户列表
      this.userlist = res.data.userlist
      this.total = res.data.total
      console.log(res)
    },
    // 监听 pagesize 改变的事件
    handleSizeChange(newSize) {
    //   console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    // 监听 页码值 改变的事件
    handleCurrentChange(newPage) {
    //   console.log(newPage)
    // 重新赋值再获取数据
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    // 监听 switch开关状态的改变
    async userStateChanged(userinfo) {
      const { data: res } = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.error('更新用户状态失败！')
      }
      this.$message.success('更新用户状态成功！')
    },
    // 监听添加用户对话框的关闭事件，并清空界面
    addDialogClosed () {
      this.$ref.addFormRef.resetFields()
    },
    // 点击按钮，添加新用户
    addUser() {
      this.$ref.addFormRef.validate(async valid => {
        // console.log(valid)
        // 如果return有问题，改为return('')
        if (!valid) return
        // 可以发起添加用户的网络请求
        const { data: res } = await this.$http.post('users', this.addForm)
        if (res.meta.status !== 201) {
          this.$message.error('添加用户失败！')
        }
        this.$message.success('添加用户成功！')
        // 隐藏添加用户的对话框
        this.addDialogVisible = false
        // 重新获取用户列表数据（刷新列表）
        this.getUserList()
      })
    },
    // 展示用户编辑的对话框
    async showEditDialog(id) {
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户信息失败！')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 监听修改用户对话框的关闭事件
    editDialogClosed() {
      this.$ref.editFormRef.resetFields()
    },
    // 修改用户信息并提交
    eitdUserInfo() {
      this.$ref.editFormRef.validate(async valid => {
        // console.log(valid)
        // 如果return有问题，改为return('')
        if (!valid) return
        // 可以发起修改用户的网络请求  修改用put 异步操作向服务器发送请求
        const { data: res } = await this.$http.put('users/', this.editForm.id, {
          email: this.editForm.email,
          mobile: this.editForm.mobile
        })
        // 提示修改是否成功  200为服务器状态码
        if (res.meta.status !== 200) {
          this.$message.error('修改用户信息失败！')
        }
        this.$message.success('修改用户信息成功！')
        // 关闭修改用户的对话框
        this.editDialogVisible = false
        // 重新获取用户列表数据（刷新列表）
        this.getUserList()
      })
    },
    // 弹窗询问用户是否删除数据
    async removeUserById(id) {
      const confirmResult = await this.$confirm('此操作将永久删除该用户，是否继续？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 如果用户确认删除，则返回值为字符串 comfirm
      // 如果用户取消删除，则返回值为字符串 cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('users/' + id)
      // 提示删除是否成功  200为服务器状态码
      if (res.meta.status !== 200) {
        this.$message.error('用户删除失败！')
      }
      this.$message.success('用户删除成功！')
      // 重新获取用户列表数据（刷新列表）
      this.getUserList()
    },
    // 展示分配角色的对话框
    async setRole(userInfo) {
      this.userInfo = userInfo
      // 在展示对话框前获取所有角色列表
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        this.$message.error('获取角色列表失败！')
      }
      // 获取角色列表数据
      this.rolesList = res.data
      this.setRoleDialogVisible = true
    },
    // 点击确定分配角色
    async saveRoleInfo() {
      if (!this.selectedRoleId) {
        this.$message.error('请选择要分配的角色！')
      }
      const { data: res } = await this.$http.put(`users/${this.userInfo.id}/role`, {
        rid: this.selectedRoleId
      })
      if (res.meta.status !== 200) {
        this.$message.error('更新角色失败！')
      }
      this.$message.success('更新角色成功！')
      // 获取角色列表数据
      this.getUserList()
      this.setRoleDialogVisible = false
    },
    // 监听分配角色对话框的关闭事件，并清空选项
    setRoleDialogClosed() {
      this.selectedRoleId = ''
      this.userInfo = {}
    }
  }
}
</script>

<style scoped>

</style>
