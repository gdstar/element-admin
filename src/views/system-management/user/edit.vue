<template>
  <div>
    <el-button icon="el-icon-back" round @click="backHandler">返回</el-button>
    <el-card>
      <div slot="header">
        <button-right>
          用户信息
          <template slot="button">
            <el-button type="primary" @click="submitHandler('form')">保存</el-button>
          </template>
        </button-right>
      </div>
      <el-form ref="form" :model="form" :rules="rules" label-width="150px">
        <el-row>
          <el-col :span="12">
            <el-form-item label="ID" prop="id">
              <el-input v-model="form.id" disabled/>
            </el-form-item>
            <el-form-item label="是否启用" prop="state">
              <el-switch v-model="form.state" :active-value="1" :inactive-value="0"/>
            </el-form-item>
            <el-form-item label="用户编号" prop="index">
              <el-input v-model="form.index"/>
            </el-form-item>
            <el-form-item label="登录ID" prop="loginName">
              <el-input v-model="form.loginName"/>
            </el-form-item>
            <el-form-item label="用户姓名" prop="name">
              <el-input v-model="form.name"/>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="照片" prop="avatar">
              <el-upload :headers="uploadAvatarHeaders" :show-file-list="false" :on-success="uploadAvatarSuccess" :on-error="uploadAvatarError" :action="uploadAvatar" class="avatar-uploader">
                <img v-if="form.avatar" :src="form.avatar" class="avatar">
                <i v-else class="el-icon-plus avatar-uploader-icon"/>
              </el-upload>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="12">
            <el-form-item label="昵称" prop="nickName">
              <el-input v-model="form.nickName"/>
            </el-form-item>
            <el-form-item label="邮箱" prop="email">
              <el-input v-model="form.email"/>
            </el-form-item>
            <el-form-item label="性别" prop="gender">
              <el-radio v-for="item in dictionaries.gender" v-model="form.gender" :key="item.key" :label="item.key">{{ item.value }}</el-radio>
            </el-form-item>
            <el-form-item label="生日" prop="birthday">
              <el-date-picker v-model="form.birthday" :picker-options="(time) => { return time.getTime() > Date.now() }" align="right" type="date"/>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="证件号码" prop="idNumber">
              <el-input v-model="form.idNumber"/>
            </el-form-item>
            <el-form-item label="电话" prop="phone">
              <el-input v-model="form.phone"/>
            </el-form-item>
            <el-form-item label="住址" prop="address">
              <el-input v-model="form.address"/>
            </el-form-item>
            <el-form-item label="标签" prop="tag" placeholder="支持英文逗号隔开">
              <el-input v-model="form.tag"/>
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="用户备注" prop="remark">
              <el-input v-model="form.remark" type="textarea"/>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
    </el-card>
    <el-card>
      <div slot="header">
        <button-right>
          密码修改
          <template slot="button">
            <el-button type="primary" @click="submitHandler('password', editUserPassword)">保存</el-button>
          </template>
        </button-right>
      </div>
      <el-form ref="password" :model="password" :rules="passwordRules" label-width="150px">
        <el-row>
          <el-col :span="12">
            <el-form-item label="新密码" prop="value">
              <el-input v-model="password.value" type="password"/>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="再次填写密码" prop="validator">
              <el-input v-model="password.validator" type="password"/>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
    </el-card>
    <el-card>
      <template slot="header">
        <select-right>
          <template slot="left">分组信息</template>
          <el-select v-model="addOption.groupId" placeholder="添加分组" clearable filterable @change="addUserGroupHandler">
            <el-option v-for="(item, index) in addOption.allGroups" :key="index" :label="item.name" :value="item.id"/>
          </el-select>
        </select-right>
      </template>
      <el-table :data="groups" border style="width: 100%">
        <el-table-column :show-overflow-tooltip="true" prop="name" label="名称"/>
        <el-table-column :show-overflow-tooltip="true" prop="remark" label="备注"/>
        <el-table-column prop="createdDate" label="创建时间" width="180" align="center">
          <template slot-scope="scope">{{ scope.row.createdDate | parseTime }}</template>
        </el-table-column>
        <el-table-column prop="modifiedDate" label="最后修改时间" width="180" align="center">
          <template slot-scope="scope">{{ scope.row.modifiedDate | parseTime }}</template>
        </el-table-column>
        <el-table-column label="操作" width="100" align="center">
          <template slot-scope="scope">
            <el-button type="warning" @click="delUserGroupHandler(scope.row.id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <el-card>
      <template slot="header">
        <select-right>
          <template slot="left">部门信息</template>
          <el-select v-model="addOption.deptId" placeholder="添加部门" clearable filterable @change="addUserDeptHandler">
            <el-option v-for="(item, index) in addOption.allDepts" :key="index" :label="item.name" :value="item.id"/>
          </el-select>
        </select-right>
      </template>
      <el-table :data="depts" border style="width: 100%">
        <el-table-column :show-overflow-tooltip="true" prop="name" label="名称"/>
        <el-table-column :show-overflow-tooltip="true" prop="remark" label="备注"/>
        <el-table-column prop="createdDate" label="创建时间" width="180" align="center">
          <template slot-scope="scope">{{ scope.row.createdDate | parseTime }}</template>
        </el-table-column>
        <el-table-column prop="modifiedDate" label="最后修改时间" width="180" align="center">
          <template slot-scope="scope">{{ scope.row.modifiedDate | parseTime }}</template>
        </el-table-column>
        <el-table-column label="操作" width="100" align="center">
          <template slot-scope="scope">
            <el-button type="warning" @click="delUserDeptHandler(scope.row.id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <el-card>
      <template slot="header">
        <select-right>
          <template slot="left">角色信息</template>
          <el-select v-model="addOption.roleId" placeholder="添加角色" clearable filterable @change="addUserRoleHandler">
            <el-option v-for="(item, index) in addOption.allRoles" :key="index" :label="item.name" :value="item.id"/>
          </el-select>
        </select-right>
      </template>
      <el-table :data="roles" border style="width: 100%">
        <el-table-column :show-overflow-tooltip="true" prop="name" label="名称"/>
        <el-table-column :show-overflow-tooltip="true" prop="remark" label="备注"/>
        <el-table-column prop="createdDate" label="创建时间" width="180" align="center">
          <template slot-scope="scope">{{ scope.row.createdDate | parseTime }}</template>
        </el-table-column>
        <el-table-column prop="modifiedDate" label="最后修改时间" width="180" align="center">
          <template slot-scope="scope">{{ scope.row.modifiedDate | parseTime }}</template>
        </el-table-column>
        <el-table-column label="操作" width="100" align="center">
          <template slot-scope="scope">
            <el-button type="warning" @click="delUserRoleHandler(scope.row.id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
import BaseEditForm from '@/views/common/mixins/BaseEditForm'
import { deepMergeLeft } from '@/utils'
import * as DeptAPI from '@/api/system-management/dept'
import * as RoleAPI from '@/api/system-management/role'
import * as GroupAPI from '@/api/system-management/group'
import * as UserAPI from '@/api/system-management/user'
import mixins from './mixins'

export default {
  mixins: [BaseEditForm, mixins],
  props: {
    detail: {
      required: false,
      type: Object,
      default: () => {}
    }
  },
  data() {
    const form = this.initForm()
    delete form.password
    const rules = this.initRules()
    delete rules.password
    rules.id = [{
      required: true, message: '编辑信息时ID不能为空', trigger: 'change'
    }]

    const validatePassword = (rule, value, callback) => {
      if (value === '') {
        callback(new Error('请再次输入密码'))
      } else if (value !== this.password.value) {
        callback(new Error('两次输入密码不一致!'))
      } else {
        callback()
      }
    }

    return {
      form: form,
      rules: rules,
      depts: [],
      groups: [],
      roles: [],

      addOption: {
        allDepts: [],
        allGroups: [],
        allRoles: [],
        deptId: null,
        roleId: null,
        groupId: null
      },

      password: {
        value: '',
        validator: ''
      },
      passwordRules: {
        value: [{
          required: true, message: '请输入密码！', trigger: 'change'
        }, {
          min: 6, max: 20, message: '长度在 6 到 20 个字符', trigger: 'change'
        }],
        validator: [{
          required: true, message: '请再次输入密码！', trigger: 'change'
        }, {
          validator: validatePassword, trigger: 'blur'
        }]
      }
    }
  },
  activated() {
    UserAPI.queryUserById(this.detail.id).then((data) => {
      deepMergeLeft(this.form, data)
      this.$nextTick(() => {
        this.$refs['form'].clearValidate()
      })
    })
    this.queryAllUserDepts()
    this.queryAllUserGroups()
    this.queryAllUserRoles()
    const params = {
      filters: [],
      sorts: []
    }
    DeptAPI.queryAllDepts(params).then(data => { this.addOption.allDepts = data })
    GroupAPI.queryAllGroups(params).then(data => { this.addOption.allGroups = data })
    RoleAPI.queryAllRoles(params).then(data => { this.addOption.allRoles = data })
  },
  methods: {
    customSubmitHandler() {
      UserAPI.editUser(this.form).then(this.submitSuccessHandler)
    },
    editUserPassword() {
      UserAPI.editUserPassword({
        id: this.form.id,
        password: this.password.value
      }).then(this.submitSuccessHandler)
    },
    customSubmitSuccessHandler() {
      this.$refs['form'].clearValidate()
      this.$refs['password'].clearValidate()
    },
    addUserDeptHandler(id) {
      if (!id) return
      const params = [{
        userId: this.detail.id,
        deptId: id
      }]
      UserAPI.addUserDept(params).then(data => {
        this.queryAllUserDepts()
        this.optionSuccessHandler()
        this.addOption.deptId = null
      })
    },
    delUserDeptHandler(id) {
      const params = {
        userId: this.detail.id,
        deptId: id
      }
      UserAPI.delUserDept(params).then(data => {
        this.queryAllUserDepts()
        this.optionSuccessHandler()
      })
    },
    addUserGroupHandler(id) {
      if (!id) return
      const params = [{
        userId: this.detail.id,
        groupId: id
      }]
      UserAPI.addUserGroup(params).then(data => {
        this.queryAllUserGroups()
        this.optionSuccessHandler()
        this.addOption.groupId = null
      })
    },
    delUserGroupHandler(id) {
      const params = {
        userId: this.detail.id,
        groupId: id
      }
      UserAPI.delUserGroup(params).then(data => {
        this.queryAllUserGroups()
        this.optionSuccessHandler()
      })
    },
    addUserRoleHandler(id) {
      if (!id) return
      const params = [{
        userId: this.detail.id,
        roleId: id
      }]
      UserAPI.addUserRole(params).then(data => {
        this.queryAllUserRoles()
        this.optionSuccessHandler()
        this.addOption.roleId = null
      })
    },
    delUserRoleHandler(id) {
      const params = {
        userId: this.detail.id,
        roleId: id
      }
      UserAPI.delUserRole(params).then(data => {
        this.queryAllUserRoles()
        this.optionSuccessHandler()
      })
    }
  }
}
</script>

<style lang="scss" scoped>
  .el-card {
    margin-top: 10px;
  }

  .avatar-uploader {

    /deep/ .el-upload {
      border: 1px dashed #d9d9d9;
      border-radius: 6px;
      cursor: pointer;
      position: relative;
      overflow: hidden;

      &:hover {
        border-color: #409EFF;
      }

      .avatar-uploader-icon {
        font-size: 28px;
        color: #8c939d;
        width: 178px;
        height: 178px;
        line-height: 178px;
        text-align: center;
      }
    }

    .avatar {
      width: 178px;
      height: 178px;
      display: block;
    }
  }
</style>
