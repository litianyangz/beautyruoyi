<template>
  <el-container>
    <el-header>
      <el-button type="primary" @click="save('form', false)">保存</el-button>
      <el-button v-if="isEdit !== true" type="primary" @click="save('form', true)">保存&继续</el-button>
    </el-header>
    <el-main>
      <el-form v-loading="loading" ref="form" :rules="rules" :model="createEditModel" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="createEditModel.roleName"/>
        </el-form-item>
        <el-form-item label="权限字符" prop="roleKey">
          <el-input v-model="createEditModel.roleKey"/>
        </el-form-item>
        <el-form-item label="显示顺序">
          <el-input-number v-model="createEditModel.roleSort" :min="0"/>
        </el-form-item>
        <el-form-item label="状态">
          <el-switch
            v-model="createEditModel.status"
            :active-value="0"
            :inactive-value="1"/>
        </el-form-item>
        <el-form-item label="备注">
          <el-input v-model="createEditModel.remark" :rows="2" type="textarea"/>
        </el-form-item>
      </el-form>
    </el-main>
  </el-container>
</template>
<script>
import DictData from '@/constants/dictdata'
import { fetchSysRoleBy, createUpdateSysRole } from '@/api/system'
const defaultForm = {
  roleId: 0,
  roleName: '',
  roleKey: '',
  roleSort: 0,
  status: DictData.EnableDisableStatus[0].value,
  remark: ''
}
export default {
  name: 'SysRoleDetail',
  props: {
    isEdit: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      createEditModel: Object.assign({}, defaultForm),
      loading: false,
      EnableDisableStatus: DictData.EnableDisableStatus,
      rules: {
        roleName: [
          { required: true, message: '必填', trigger: 'blur' }
        ],
        roleKey: [
          { required: true, message: '必填', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    if (this.isEdit) {
      const id = this.$route.params && this.$route.params.id
      fetchSysRoleBy(id).then(res => {
        this.createEditModel = res.data
        console.log('create edit model' + JSON.stringify(this.createEditModel))
      }
      ).catch(ex => {
        this.loading = false
      });
    } else {
      this.createEditModel = Object.assign({}, defaultForm)
      this.loading = false
    }
    // Why need to make a copy of this.$route here?
    // Because if you enter this page and quickly switch tag, may be in the execution of the setTagsViewTitle function, this.$route is no longer pointing to the current page
    // https://github.com/PanJiaChen/vue-element-admin/issues/1221
    // this.tempRoute = Object.assign({}, this.$route)
  },
  methods: {
    save(formName, continueEdit) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          this.loading = true;
          var postData = Object.assign({}, this.createEditModel)
          createUpdateSysRole(postData).then(res => {
            console.log('create success');
            this.loading = false
            this.$notify({
              message: '操作成功',
              type: 'success'
            });
            if (continueEdit === true) {
              this.createEditModel = Object.assign({}, defaultForm)
            } else {
              this.$store.dispatch('delView', this.$route).then(res => {
                this.$router.push('/system/role/list')
              });
            }
          }).catch(ex => {
            console.log('create dept error');
            this.loading = false
          })
        }
      });
    }
  }
}
</script>
