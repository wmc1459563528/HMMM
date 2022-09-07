<template>
  <div class="dashboard-container">
    <div class="app-container">
      <el-card shadow="never">
        <!-- 搜索 -->
        <el-form :model="form" ref="form">
          <div class="filter-container">
            <el-row :gutter="20">
              <el-col :span="6">
                <el-form-item label="标签名称：" prop="tags">
                  <el-input
                    @keyup.enter="hQuery"
                    style="width: 180px;"
                    placeholder="请输入"
                    class="filter-item"
                    v-model="form.tags"
                  ></el-input>
                </el-form-item>
              </el-col>
              <el-col :span="8">
                <el-form-item label="城市">
                  <MyCity v-model="form.provAndCity"/>
                </el-form-item>
              </el-col>
              <el-col :span="6">
                <el-form-item label="企业简称：" prop="shortName">
                  <el-input v-model.trim="form.shortName" style="width: 180px;"></el-input>
                </el-form-item>
              </el-col>
              <el-col :span="4">
                <el-form-item label="状态：" prop="state">
                  <el-select
                    class="filter-item"
                    style="width: 100px;"
                    v-model="form.state"
                    clearable
                  >
                    <el-option
                      v-for="item in status"
                      :key="item.value"
                      :label="item.label"
                      :value="item.value"
                    ></el-option>
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="6">
                <el-form-item>
                  <el-button
                    :loading="isLoading"
                    size="small"
                    type="primary"
                    @click="hQuery"
                  >查询</el-button>
                  <el-button
                  size="small"
                  @click="hResetForm">重置</el-button>
                </el-form-item>
              </el-col>
            </el-row>

            <el-button
              class="filter-item fr"
              size="small"
              style="margin-left: 10px;"
              @click="hAddComponey"
              type="primary"
              icon="el-icon-edit"
            >新增公司</el-button>
          </div>
        </el-form>
        <!-- 数据记录 -->
        <el-alert
          v-if="cResultTip"
          :title="cResultTip"
          type="info"
          class="alert"
          :closable="false"
          show-icon
        ></el-alert>
        <!-- end -->
        <!-- 数据列表 -->
        <el-table
          :data="companys"
          v-loading="isLoading"
          element-loading-text="给我一点时间"
          fit
          highlight-current-row
          style="width: 100%"
          border
        >
          <el-table-column align="center" label="序号" width="70">
            <template slot-scope="scope">
              <span>{{scope.row.id}}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="企业编号">
            <template slot-scope="scope">
              <span>{{scope.row.number}}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="企业简称">
            <template slot-scope="scope">
              <span>{{scope.row.shortName}}</span>
            </template>
          </el-table-column>
          <el-table-column label="标签">
            <template slot-scope="scope">
              <span>{{scope.row.tags}}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="创建者">
            <template slot-scope="scope">
              <span>{{scope.row.creatorID}}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="创建日期">
            <template slot-scope="scope">
              <span>{{scope.row.addDate}}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="备注">
            <template slot-scope="scope">
              <span>{{scope.row.remarks.substring(0,3)}}</span>
            </template>
          </el-table-column>
          <el-table-column align="center" label="状态" width="80">
            <template slot-scope="scope">
              <span v-if="scope.row.state==1">启用</span>
              <span v-else>禁用</span>
            </template>
          </el-table-column>
          <!-- 操作按钮 -->
          <el-table-column
            align="center"
            label="操作"
            width="280"
            class-name="small-padding fixed-width"
          >
            <template slot-scope="scope">
              <el-button type="primary" size="mini" @click="hEdit(scope.row.id)">编辑</el-button>
              <el-button type="primary" size="mini" @click="hSwitchStatus(scope.row)">
                <span >{{scope.row.state ===0 ? '启用' : '禁用'}}</span>
              </el-button>
              <el-button
                v-if="scope.row.status!='deleted'"
                size="mini"
                type="danger"
                @click="hDelete(scope.row.id)"
              >删除</el-button>
            </template>
          </el-table-column>
        </el-table>
        <!-- end -->
        <!-- 分页 -->
        <div class="pagination" v-show="total">
          <div class="pages">
            <el-pagination
              background
              @size-change="hSizeChange"
              @current-change="hCurrentChange"
              :current-page="Number(form.page)"
              :total="Number(total)"
              :page-size="Number(form.pagesize)"
              :page-sizes="[10,20,30, 50]"
              layout="sizes, prev, pager, next, jumper"
            ></el-pagination>
          </div>
        </div>
        <!-- end -->
      </el-card>

      <!-- 新增标签弹层

        把对话框留在父组件中，把对话框中的核心内容提出来，封装成一个组件。

        为什么不把对话框一起封装？
        (1) 便于复用。如果与对话框绑定了，则后期如果要单独拿做一个页面，则无法复用。
        (2) 对话框留在父组件，操作更方便。
      -->
      <el-dialog :title="cTitle"
        :visible.sync="isShowDialog"
        v-if="isShowDialog"
        >
        <CompanysAdd ref="editUser"
        :companyId="curCompanyId"
        @close="isShowDialog=false"
        @refresh="loadCompanys"></CompanysAdd>

      </el-dialog>
    </div>
  </div>
</template>

<script>
import { status } from '@/api/hmmm/constants'
import { list as getCompanys, remove, disabled } from '@/api/hmmm/companys'
import CompanysAdd from './companys-add'
import MyCity from './citys'
export default {
  name: 'CompanysList',
  components: {
    CompanysAdd,
    MyCity
  },
  data () {
    return {
      companys: [],
      total: 0,

      curCompanyId: null, // 区别于是否是编辑，还是添加

      isLoading: false,
      isShowDialog: false,

      // 查询表单中的参数
      form: {
        shortName: '',
        state: null,
        tags: '',
        provAndCity: {
          province: '',
          city: ''
        },
        page: 1,
        pagesize: 10
      }
    }
  },
  // 创建完毕,做一次查询
  created () {
    this.loadCompanys()
  },
  computed: {
    status () {
      return status
    },
    cResultTip () {
      return `共 ${this.total} 条记录`
    },
    cTitle () {
      return this.curCompanyId ? '编辑公司' : '添加新公司'
    }
  },
  methods: {
    buildParams () {
      const paramsObj = {}
      const { page, pagesize, provAndCity, tags, state } = this.form
      // 如果某一个参数是null,则axios就不会发这个参数
      // 如果某一个参数是""，axios是会发出这个参数
      paramsObj.page = page || null
      paramsObj.pagesize = pagesize || null
      paramsObj.tags = tags || null
      paramsObj.province = provAndCity.city || null
      paramsObj.city = provAndCity.city || null
      paramsObj.state = state === '' ? null : state
      return paramsObj
    },
    // 获取列表数据
    async loadCompanys () {
      this.isLoading = true
      // 构建查询参数
      const paramsObj = this.buildParams()

      try {
        const { data: res } = await getCompanys(paramsObj)
        // 获取数据给companys
        this.companys = res.items
        // 获取总条数
        this.total = res.counts

        this.isLoading = false
      } catch (err) {
        console.log(err)
        this.isLoading = true
      }
    },
    // 重置整个表单
    hResetForm () {
      // el 表单组件上有一个resetFields方法
      this.$refs.form.resetFields()

      // 由于provAndCity传给了自定义组件，它们并不是在当前表单中直接
      // 使用的，所以，这里要去做手动清空
      this.form.provAndCity.province = ''
      this.form.provAndCity.city = ''
    },
    // 搜索信息
    hQuery () {
      // 重置页码
      this.form.page = 1
      // 重发请求
      this.loadCompanys()
    },
    // 更改每页显示的条数
    hSizeChange (val) {
      this.form.pagesize = val

      if (this.form.page === 1) {
        this.loadCompanys()
      }
    },
    // 点击某一个页码
    hCurrentChange (curPage) {
      this.form.page = curPage
      this.loadCompanys()
    },

    // 新增用户、编辑用户
    hAddComponey () {
      this.curCompanyId = ''
      this.isShowDialog = true
    },
    hEdit (id) {
      this.curCompanyId = id
      this.isShowDialog = true
    },
    async doDelete (id) {
      try {
        await remove({ id })
        this.$message.success('成功删除了用户!')
        this.loadCompanys()
      } catch (err) {
        this.$message.error('删除失败!')
      }
    },
    // 删除
    hDelete (userId) {
      this.$confirm('此操作将永久删除用户 ' + ', 是否继续?', '提示', {
        type: 'warning'
      })
        .then(() => {
          this.doDelete(userId)
        })
        .catch(() => {
          this.$message.info('已取消操作!')
        })
    },
    async doSwitch (row) {
      var data = {
        id: row.id,
        state: Number(!row.state)
      }
      console.log(data)
      await disabled(data)
        .then(response => {
          this.$message.success('操作成功!')
          this.loadCompanys(this.form)
        })
        .catch(response => {
          this.$message.error(status + '失败!')
        })
    },
    // 启用、禁用
    hSwitchStatus (row) {
      const tip = '已成功' + (row.state ? '禁用' : '启用') + ', 是否继续?'
      this.$confirm(tip, '提示', {
        type: 'warning'
      })
        .then(() => {
          this.doSwitch(row)
        })
        .catch(() => {
          this.$message.info('已取消操作!')
        })
    }
  }
}
</script>
<style rel="stylesheet/scss" lang="scss" scoped>
.alert {
  margin: 10px 0px;
}
.pagination {
  margin-top: 10px;
}
</style>

<style>
.el-table th {
  background-color: #fafafa;
}
.el-table th.is-leaf {
  border-bottom: 2px solid #e8e8e8;
}
.disabled td {
  background-color: #f9f9f9;
  color: #c1c1c1;
}
.disabled .el-button--primary,
.disabled .el-button--danger {
  background-color: #dbdada;
  border: 1px solid #dbdada;
  color: #999;
  cursor: not-allowed;
}
</style>
