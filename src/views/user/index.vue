<template>
  <div class="app-container">
    <el-button type="primary" @click="handleAddUser">新建用户</el-button>

    <el-table :data="usersList" style="width: 100%;margin-top:30px;" size="mini" border>
      <el-table-column align="center" label="用户名" width="220">
        <template slot-scope="scope">
          {{ scope.row.username }}
        </template>
      </el-table-column>
      <el-table-column align="center" label="姓名" width="220">
        <template slot-scope="scope">
          {{ scope.row.name }}
        </template>
      </el-table-column>
      <el-table-column align="center" label="电话">
        <template slot-scope="scope">
          {{ scope.row.phone }}
        </template>
      </el-table-column>
      <el-table-column align="center" label="权限">
        <template slot-scope="scope">
          {{ scope.row.roleName }}
        </template>
      </el-table-column>
      <el-table-column align="center" label="创建日期">
        <template slot-scope="scope">
          {{ scope.row.createTime }}
        </template>
      </el-table-column>
      <el-table-column align="center" label="操作">
        <template slot-scope="scope">
          <el-button type="primary" size="mini" @click="handleEdit(scope)">修改</el-button>
          <el-button type="danger" size="mini" @click="handleDelete(scope)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-dialog :visible.sync="dialogVisible" :title="dialogType==='edit'?'修改用户':'新建用户'">
      <el-form ref="userForm" :model="user" label-width="80px" :rules="userRules" label-position="right" size="mini" :inline="true">
        <el-form-item label="用户名" required>
          <el-input v-model="user.username" placeholder="用户名" />
        </el-form-item>
        <el-form-item label="密码">
          <el-input v-model="user.password" placeholder="密码" />
        </el-form-item>
        <el-form-item label="姓名">
          <el-input v-model="user.name" placeholder="姓名" />
        </el-form-item>
        <el-form-item label="性别">
          <!-- <el-input v-model="user.username" /> -->
          <el-radio v-model="user.gender" label="M">男</el-radio>
          <el-radio v-model="user.gender" label="F">女</el-radio>
        </el-form-item>
        <el-form-item label="电话">
          <el-input v-model="user.phone" placeholder="电话号码" />
        </el-form-item>
        <el-form-item label="地址">
          <el-input v-model="user.address" placeholder="地址" />
        </el-form-item>
        <el-form-item label="权限">
          <el-select v-model="user.roleId" placeholder="请选择" @change="handleUserRoleChange(user)">
            <el-option
              v-for="item in roleOptions"
              :key="item.id"
              :label="item.name"
              :value="item.id"
            />
          </el-select>
        </el-form-item>
      </el-form>
      <div style="text-align:right;">
        <el-button type="danger" @click="dialogVisible=false">取消</el-button>
        <el-button type="primary" @click="confirmUser">保存</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { deepClone } from '@/utils'
import { getAllUsers, addUser, updateUser, deleteUser } from '@/api/user'
import { getAllRoles } from '@/api/role'

const defaultUser = {
  username: '',
  name: '',
  gender: '',
  phone: '',
  address: '',
  roleId: '',
  roleName: '',
  introduction: ''
}

export default {
  data() {
    return {
      user: Object.assign({}, defaultUser),
      usersList: [],
      dialogVisible: false,
      dialogType: 'new',
      roleOptions: [],
      userRules: {
        // username: [{ required: true, message: '请输入用户名', trigger: 'blur' }],
        // password: [{ required: true, trigger: 'blur', validator: validatePassword }]
        // name: [],
      }
    }
  },
  created() {
    this.getUsers()
    this.getRoles()
  },
  methods: {
    async getUsers() {
      const res = await getAllUsers()
      this.usersList = res.data
    },
    async getRoles() {
      const res = await getAllRoles()
      this.roleOptions = res.data
    },
    handleAddUser() {
      this.user = Object.assign({}, defaultUser)
      this.dialogType = 'new'
      this.dialogVisible = true
    },
    handleEdit(scope) {
      this.dialogType = 'edit'
      this.dialogVisible = true
      this.checkStrictly = true
      this.user = deepClone(scope.row)
    },
    handleUserRoleChange(user) {
      if (!user.roleId) {
        user.roleName = ''
      } else {
        const selectedRole = this.roleOptions.find(option => option.id === user.roleId)
        user.roleName = selectedRole.name
      }
    },
    handleDelete({ $index, row }) {
      this.$confirm('确定要将该用户删除吗?', '警告', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async() => {
          await deleteUser(row.id)
          this.usersList.splice($index, 1)
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        })
        .catch(err => { console.error(err) })
    },
    async confirmUser() {
      this.$refs.userForm.validate(async(valid) => {
        if (valid) {
          const isEdit = this.dialogType === 'edit'

          if (isEdit) {
            await updateUser(this.user)
          } else {
            await addUser(this.user)
          }
          this.getUsers()
          this.dialogVisible = false
          this.$notify({
            title: '成功',
            dangerouslyUseHTMLString: true,
            message: `
                <div>添加用户成功</div>
              `,
            type: 'success'
          })
        } else {
          console.log('validate error!!')
          return false
        }
      })
    }
  }
}
</script>

<style lang="scss" scoped>
// .app-container {}
</style>
