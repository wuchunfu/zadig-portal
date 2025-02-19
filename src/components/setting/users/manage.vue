<template>
  <div class="users-overview-container">
    <!--start of add user dialog-->
    <el-dialog :title="$t('sysSetting.users.addUser')" custom-class="create-user-dialog" :close-on-click-modal="false" :visible.sync="dialogAddUserVisible">
      <el-form
        :model="addUser"
        @submit.native.prevent
        :rules="addUserRule"
        ref="addUserForm"
        label-position="left"
        label-width="100px"
        class="primary-form"
      >
        <el-form-item :label="$t('sysSetting.users.username')" prop="account">
          <el-input size="small" v-model="addUser.account"></el-input>
        </el-form-item>
        <el-form-item :label="$t('sysSetting.users.password')" prop="password">
          <el-input size="small" v-model="addUser.password"></el-input>
        </el-form-item>
        <el-form-item :label="$t('sysSetting.users.mail')" prop="email">
          <el-input size="small" v-model="addUser.email"></el-input>
        </el-form-item>
        <el-form-item :label="$t('sysSetting.users.nickname')" prop="name">
          <el-input size="small" v-model="addUser.name"></el-input>
        </el-form-item>
        <el-form-item :label="$t('sysSetting.users.phone')" prop="phone">
          <el-input size="small" v-model="addUser.phone"></el-input>
        </el-form-item>
        <el-form-item :label="$t('sysSetting.users.role')" prop="isAdmin">
          <el-select size="small" v-model="addUser.isAdmin" multiple :placeholder="$t('sysSetting.users.selectRole')">
            <el-option :label="item.name" :value="item.name"  v-for="item in roleList" :key="item.desc">
          </el-option>
        </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" native-type="submit" size="small" @click="addUserOperation" class="start-create">{{$t(`global.confirm`)}}</el-button>
        <el-button plain native-type="submit" size="small" @click="dialogAddUserVisible = false">{{$t(`global.cancel`)}}</el-button>
      </div>
    </el-dialog>
    <!--end of add user dialog-->
    <div class="search-user">
      <el-row :gutter="10">
        <el-col :span="10">
          <div class="search-member">
            <span class="text-title">{{$t('sysSetting.users.searchUser')}}</span>
            <el-button v-if="!searchInputVisible" size="small" @click="searchInputVisible=true" plain type="primary" icon="el-icon-search"></el-button>
            <transition name="fade">
              <el-input
                v-if="searchInputVisible"
                size="small"
                v-model.lazy="searchUser"
                :placeholder="$t('sysSetting.users.inputNickname')"
                autofocus
                clearable
                prefix-icon="el-icon-search"
              ></el-input>
            </transition>
          </div>
        </el-col>
        <el-col :span="6">
          <el-button @click="dialogAddUserVisible=true" size="small" plain type="primary">{{$t('sysSetting.users.addUser')}}</el-button>
        </el-col>
        <el-col :span="6">
          <div style="width: 100%; line-height: 32px;">
            <span class="text-title">{{$t('sysSetting.users.userRegistration')}}</span>
            <el-switch v-model="registrationStatus"
                       @change="changeRegistration"
                       active-color="#0066ff">
            </el-switch>
          </div>
        </el-col>
      </el-row>
    </div>
    <div
      v-loading="loading"
      :element-loading-text="$t(`global.loading`)"
      element-loading-spinner="iconfont iconfont-loading icongeren"
      class="users-container"
    >
      <el-table :data="users" style="width: 100%;">
        <el-table-column :label="$t('sysSetting.users.users')">
          <template slot-scope="scope">
            <div class="name-listing-details">
              <!-- Logo -->
              <div class="avator">
                <span class="iconfont iconvery-user"></span>
              </div>
              <!-- Details -->
              <div class="name-listing-description">
                <h3 class="name-listing-title">
                  {{ scope.row.name ? `${scope.row.name}(${scope.row.account})`: scope.row.account }}
                  <el-tag size="mini" v-if="scope.row.admin"  effect="plain">{{$t('sysSetting.users.admin')}}</el-tag>
                </h3>
                <!-- Name Listing Footer -->
                <div class="name-listing-footer">
                  <ul>
                    <li v-if="scope.row.email">
                      <i class="icon el-icon-message"></i>
                      <a :href="`mailto:${scope.row.email}`">{{scope.row.email}}</a>
                    </li>
                    <li v-if="scope.row.phone">
                      <i class="icon el-icon-mobile"></i>
                      <a :href="`tel:${scope.row.phone}`">{{scope.row.phone}}</a>
                    </li>
                  </ul>
                </div>
              </div>
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="last_login_time" :label="$t('sysSetting.users.loginInfo')" width="180">
          <template slot-scope="scope">
            <span v-if="scope.row.last_login_time">{{$utils.convertTimestamp(scope.row.last_login_time)}}</span>
            <span v-else>{{$t('sysSetting.users.notLoggedIn')}}</span>
          </template>
        </el-table-column>
        <el-table-column :label="$t('sysSetting.users.source')" width="120">
          <template slot-scope="scope">
            <span v-if="scope.row.identity_type" class="origin">
              <i class="iconfont type" :class="'icon'+scope.row.identity_type"></i>
              {{identityTypeMap[scope.row.identity_type]}}
            </span>
          </template>
        </el-table-column>
        <el-table-column :label="$t(`global.operation`)" width="180">
          <template slot-scope="scope">
            <el-button @click="editUserInfo(scope.row)" type="primary" size="mini" plain>{{$t(`global.edit`)}}</el-button>
            <el-button @click="deleteUser(scope.row)" type="danger" size="mini" plain>{{$t(`global.delete`)}}</el-button>
          </template>
        </el-table-column>
      </el-table>
      <!--start of page-divide -->
      <div class="user-table-pagination">
        <el-pagination
          background
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="currentPageList"
          :page-sizes="[10, 20, 30, 40]"
          :page-size="userPageSize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="totalUser"
        ></el-pagination>
      </div>
      <!--page divide-->
    </div>
    <EditUserRole ref="editUserInfo" :editUser="editUser" @refreshUserList="getUsers(userPageSize, currentPageList, searchUser)" />
  </div>
</template>

<script>
import {
  addUserAPI,
  getUsersAPI,
  deleteUserAPI,
  updateSystemRoleBindingsAPI,
  checkRegistrationAPI,
  changeRegistrationAPI,
  getRoleListAPI
} from '@api'
import EditUserRole from './editUserInfo.vue'
export default {
  components: {
    EditUserRole
  },
  data () {
    return {
      users: [],
      addUser: {
        account: '',
        name: '',
        email: '',
        password: '',
        phone: '',
        isAdmin: false
      },
      editUser: {
        account: ''
      },
      searchUser: '',
      totalUser: 0,
      userPageSize: 10,
      currentPageList: 1,
      dialogEditRoleVisible: false,
      dialogAddUserVisible: false,
      searchInputVisible: true,
      registrationStatus: false,
      loading: true,
      roleList: []
    }
  },
  computed: {
    identityTypeMap () {
      return {
        github: 'GitHub',
        system: this.$t('sysSetting.users.identityTypeSystem'),
        ldap: 'OpenLDAP',
        oauth: 'OAuth'
      }
    },
    addUserRule () {
      return {
        account: [
          {
            type: 'string',
            required: true,
            message: this.$t('sysSetting.users.inputUsername'),
            trigger: 'blur'
          }
        ],
        email: [
          {
            type: 'string',
            required: true,
            message: this.$t('sysSetting.users.inputMail'),
            trigger: 'blur'
          },
          {
            type: 'email',
            message: this.$t('sysSetting.users.checkMail'),
            trigger: ['blur', 'change']
          }
        ],
        name: [
          {
            type: 'string',
            required: true,
            message: this.$t('sysSetting.users.inputNickname'),
            trigger: 'blur'
          }
        ],
        password: [
          {
            type: 'string',
            required: true,
            message: this.$t('sysSetting.users.inputPassword'),
            trigger: 'blur'
          }
        ]
      }
    }
  },
  methods: {
    editUserInfo (user) {
      user.isAdmin = user.system_role_bindings.map(item => item.role)
      this.editUser = user
      this.$refs.editUserInfo.dialogEditRoleVisible = true
    },
    submit () {
      this.$refs.form.validate(valid => {
        if (valid) {
          this.upsertRole()
        }
      })
    },
    async getUsers (page_size = 0, page_index = 0, keyword = '') {
      this.loading = true
      const payload = {
        page: page_index,
        per_page: page_size,
        name: keyword
      }
      const usersData = await getUsersAPI(payload).catch(error =>
        console.log(error)
      )
      if (usersData) {
        this.totalUser = usersData.total_count
        this.users = usersData.users
      }
      this.loading = false
    },
    deleteUser (row) {
      this.$confirm(
        this.$t('sysSetting.users.deleteUserTip', { type: this.identityTypeMap[row.identity_type], name: row.name ? row.name : row.account }),
        this.$t('sysSetting.users.tip'),
        {
          confirmButtonText: this.$t(`global.confirm`),
          cancelButtonText: this.$t(`global.cancel`),
          type: 'warning'
        }
      )
        .then(() => {
          deleteUserAPI(row.uid).then(res => {
            this.$message({
              type: 'success',
              message: this.$t('sysSetting.users.userHasBeenDeleted')
            })
            this.getUsers(
              this.userPageSize,
              this.currentPageList,
              this.searchUser
            )
          })
        })
        .catch(error => {
          console.log(error)
          this.$message({
            type: 'info',
            message: this.$t('sysSetting.users.cancelDelete')
          })
        })
    },
    addUserOperation () {
      this.$refs.addUserForm.validate(valid => {
        if (valid) {
          const params = []
          const payload = this.addUser
          addUserAPI(payload).then(async res => {
            this.dialogAddUserVisible = false
            if (payload.isAdmin) {
              this.addUser.isAdmin.forEach((item, index) => {
                const obj = {
                  name: `user:${res.uid},role:${item}`,
                  role: item,
                  uid: res.uid
                }
                params.push(obj)
              })

              await updateSystemRoleBindingsAPI(res.uid, params).catch(error =>
                console.log(error)
              )
            }
            this.$refs.addUserForm.resetFields()
            this.getUsers(
              this.userPageSize,
              this.currentPageList,
              this.searchUser
            )
            this.$message({
              type: 'success',
              message: this.$t('sysSetting.users.addImageSuccess')
            })
          })
        } else {
          return false
        }
      })
    },
    checkRegistration () {
      checkRegistrationAPI().then(res => {
        this.registrationStatus = res.enabled
      })
    },
    changeRegistration (val) {
      const payload = {
        name: 'RegisterTrigger',
        enabled: val
      }
      changeRegistrationAPI(payload).then(res => {
        this.checkRegistration()
        this.$message({
          type: 'success',
          message: this.$t('sysSetting.users.changeUserRegistrationStatusSuccess')
        })
      })
    },
    handleSizeChange (val) {
      this.userPageSize = val
      this.getUsers(this.userPageSize, this.currentPageList, this.searchUser)
    },
    handleCurrentChange (val) {
      this.currentPageList = val
      this.getUsers(this.userPageSize, this.currentPageList, this.searchUser)
    },
    async getRoleList (page_size = 0, page_index = 0) {
      const payload = {
        page: page_index,
        per_page: page_size
      }
      const res = await getRoleListAPI(payload).catch(error => {
        console.log(error)
      })

      if (res) {
        this.roleList = res
      }
    }
  },
  watch: {
    searchUser: function (val, oldVal) {
      this.getUsers(this.userPageSize, this.currentPageList, val)
    },
    dialogAddUserVisible: function (val, oldVal) {
      if (val) {
        this.getRoleList()
      }
    }
  },
  created () {
    this.getUsers(this.userPageSize, this.currentPageList, this.searchUser)
    this.checkRegistration()
  }
}
</script>

<style lang="less" scoped>
.users-overview-container {
  position: relative;
  flex: 1;
  overflow-x: hidden;
  overflow-y: auto;
  font-size: 13px;

  .users-container {
    .origin {
      .type {
        font-size: 20px;
      }
    }

    .name-listing-details {
      top: 0;
      display: flex;
      flex-wrap: nowrap;
      align-items: center;
      padding: 0;

      .avator span {
        position: relative;
        margin-right: 12px;
        color: @themeColor;
        font-size: 25px;
      }

      .name-listing-description {
        display: flex;
        flex-direction: column;

        .name-listing-title {
          margin: 0;
          color: #333;
          font-weight: 300;
          font-size: 16px;
        }
      }

      .name-listing-footer {
        position: relative;
        padding: 0;
        background-color: transparent;

        ul {
          display: flex;
          flex-direction: column;
          margin: 0;
          padding: 0;
          list-style: none;

          li {
            display: flex;
            align-items: center;
            color: #777;
            font-size: 14px;

            .icon {
              margin-right: 2px;
            }

            a {
              white-space: nowrap;
            }
          }
        }
      }
    }

    .user-table-pagination {
      margin-top: 25px;
    }
  }

  .search-user {
    margin-top: 10px;
    margin-bottom: 15px;

    .text-title {
      margin-right: 5px;
      color: rgba(0, 0, 0, 0.65);
    }

    .search-member {
      .el-input {
        width: calc(~'100% - 80px');
      }
    }
  }
}
</style>

<style lang="less">
.create-user-dialog {
  width: 600px;

  .el-dialog__header {
    padding: 15px;
    text-align: center;
    border-bottom: 1px solid #e4e4e4;

    .el-dialog__close {
      font-size: 10px;
    }
  }

  .el-dialog__body {
    padding: 30px 48px 0;

    .el-form.primary-form {
      .el-form-item {
        margin-bottom: 16px;
      }
    }
  }
}
</style>
