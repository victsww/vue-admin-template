<!--
 * @Author        : fineemb
 * @Github        : https://github.com/fineemb
 * @Description   :  f
 * @Date          : 2020-03-03 22:04:12
 * @LastEditors   : fineemb
 * @LastEditTime  : 2020-03-05 12:20:07
 -->
<template>
  <div class="app-container">
    <el-form ref="form" :model="form" :rules="rules">
      <div class="filter-container">
        <el-form-item prop="name" class="filter-item">
          <el-input v-model="form.name" placeholder="小爱将用这个名称唤醒你的设备" />
        </el-form-item>
        <el-form-item prop="type" class="filter-item">
          <el-select v-model="form.type" placeholder="选择一个设备类型">
            <el-option label="灯" value="fine.light.light" />
            <el-option label="插座" value="fine.plug.dc1" />
            <el-option label="窗帘" value="fine.curtain.fv1" />
          </el-select>
        </el-form-item>
        <el-button type="primary" icon="el-icon-circle-plus" class="filter-item" @click.native="onSubmit('form')"> 创建</el-button>
        <el-button class="filter-item" @click="onCancel('form')">取消</el-button>
      </div>
    </el-form>

    <el-table
      v-loading="listLoading"
      :data="list"
      element-loading-text="Loading"
      border
      fit
      highlight-current-row
    >
      <el-table-column align="center" label="设备ID" width="220">
        <template slot-scope="scope">
          {{ scope.row.did }}
        </template>
      </el-table-column>

      <el-table-column label="设备名称" width="300px" align="center" sortable prop="name">
        <template slot-scope="scope">
          <template v-if="scope.row.edit">
            <el-input v-model="scope.row.name" class="edit-input" size="small" />
            <el-button
              class="cancel-btn"
              size="small"
              icon="el-icon-refresh"
              type="warning"
              @click="cancelEdit(scope.row)"
            >
              取消
            </el-button>
          </template>
          <span v-else>{{ scope.row.name }}</span>
        </template>
      </el-table-column>

      <el-table-column label="设备类型" width="110" align="center" sortable prop="type">
        <template slot-scope="scope">
          <span>{{ scope.row.type }}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" prop="created_at" label="创建日期" width="200">
        <template slot-scope="scope">
          <i class="el-icon-time" />
          <span>{{ scope.row.display_time }}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" label="操作" width="220">
        <template slot-scope="scope">
          <el-button
            v-if="scope.row.edit"
            type="success"
            size="small"
            icon="el-icon-circle-check"
            @click="confirmEdit(scope.row)"
          >
            确定
          </el-button>
          <el-button
            v-else
            type="primary"
            size="small"
            icon="el-icon-edit"
            @click="scope.row.edit=!scope.row.edit"
          >
            修改
          </el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            @click="delDevice(scope.row)"
          >删除</el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import { getList, addDevice, delDevice, upDataDevice } from '@/api/devices'

export default {
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'gray',
        deleted: 'danger'
      }
      return statusMap[status]
    }
  },
  data() {
    return {
      list: null,
      listLoading: true,
      form: {
        name: '',
        type: ''
      },
      rules: {
        name: [{ required: true, message: '请输入一个名称', trigger: 'blur' }],
        type: [{ required: true, message: '请选择一个正确的设备类型', trigger: 'change' }]
      }
    }
  },
  created() {
    this.fetchData()
  },
  methods: {
    fetchData() {
      this.listLoading = true
      getList().then(response => {
        const items = response.data.items
        this.list = items.map(v => {
          this.$set(v, 'edit', false) // https://vuejs.org/v2/guide/reactivity.html
          v.originalTitle = v.title //  will be used when user click the cancel botton
          return v
        })
        this.listLoading = false
      })
    },
    cancelEdit(row) {
      row.title = row.originalTitle
      row.edit = false
      this.$message({
        message: '名称取消修改',
        type: 'warning'
      })
    },
    confirmEdit(row) {
      row.edit = false
      row.originalTitle = row.title
      const newdata = {
        did: row.did,
        name: row.title
      }
      upDataDevice(newdata).then(response => {
        this.listLoading = false
        this.$message({
          message: 'ID为 ' + row.did + ' 的设备名称更改成功',
          type: 'success'
        })
      })
    },
    delDevice(row) {
      this.$confirm('此操作将永久删除该设备, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.listLoading = true
        const data = {
          did: row.did
        }
        delDevice(data).then(response => {
          this.listLoading = false
          this.$message({
            message: '成功删除ID为 ' + row.did + ' 的设备',
            type: 'success'
          })
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    onSubmit(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          this.listLoading = true
          addDevice(this.form).then(response => {
            this.listLoading = false
            this.$message('设备创建成功!')
          })
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    onCancel(formName) {
      this.$refs[formName].resetFields()
    }
  }
}
</script>
<style scoped>
.edit-input {
  padding-right: 100px;
}
.cancel-btn {
  position: absolute;
  right: 15px;
  top: 10px;
}
.filter-container {
    padding-bottom: 10px;
    display: flex;
}
.filter-container .filter-item {
    display: inline-block;
    vertical-align: middle;
    margin-bottom: 10px;
    margin-right: 5px;
}
.el-input {
  width: 300px;
}

</style>
