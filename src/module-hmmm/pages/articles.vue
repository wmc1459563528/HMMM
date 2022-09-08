<template>
  <div class="dashboard-container">
    <div class="app-container">
      <el-card shadow="never">
          <!-- 顶部表单部分 -->
        <el-row>
          <el-col :span="22">
            <div class="grid-content bg-purple">
              <!-- :inline="true" 让输入框在同一行 -->
              <el-form :inline="true" :model="form" class="demo-form-inline">
                <el-form-item label="关键字" class="key-word">
                  <el-input v-model="form.keyword"></el-input>
                </el-form-item>
                <el-form-item label="状态" class="state">
                  <el-select v-model="form.state" placeholder="请选择">
                    <el-option label="启用" value="1"></el-option>
                    <el-option label="禁用" value="0"></el-option>
                  </el-select>
                </el-form-item>
                <el-form-item>
                  <el-button size="small" @click="handleClear">清除</el-button>
                  <el-button size="small" type="primary">搜索</el-button>
                </el-form-item>
              </el-form>
            </div>
          </el-col>
          <el-col :span="2">
            <div class="grid-content bg-purple">
              <el-button size="small" type="success" icon="el-icon-edit"
                >新增技巧</el-button
              >
            </div>
          </el-col>
        </el-row>
        <!-- 数据提示 -->
        <el-row>
          <el-col :span="24">
            <!-- ${} 替换字符串 :closable='false'不可关闭 -->
            <el-alert
              :title="`数据一共${total}条`"
              type="info"
              show-icon
              :closable="false"
            ></el-alert>
          </el-col>
        </el-row>
        <!-- 文章渲染 -->
        <el-row>
          <el-col :span="24">
            <el-table :data="ArticlesList" style="width: 100%">
              <el-table-column type="index" label="序号" :index="indexMethod" />
              <el-table-column prop="title" label="文章标题" width="300" />
              <el-table-column prop="visits" label="阅读数" width="150" />
              <el-table-column prop="username" label="录入人" width="150" />
              <el-table-column label="录入时间" width="250">
                <!--
                  如果和父组件中先用到子组件中的数据 就需要用到 slot-scope
                  scope可以获取到 row, column, $index 和 store（table 内部的状态管理）的数据
                   -->
                <template slot-scope="scope">
                  <!-- 利用管道符来修改时间格式 -->
                  {{ scope.row.createTime|parseTimeByString}}
                </template>
              </el-table-column>
              <el-table-column  label="状态" width="100">
                <template slot-scope="scope">
                  {{scope.row.state === 1 ? '已启用': '未启用'}}
                </template>
              </el-table-column>
              <el-table-column label="操作">
                <template slot-scope="scope">
                  <el-button type="text">预览</el-button>
                  <el-button type="text" @click="handleArticlesState(scope.row)">{{scope.row.state === 1 ? '启用': '禁用'}}</el-button>
                  <el-button type="text" :disabled="scope.row.state === 1">修改</el-button>
                  <el-button type="text" :disabled="scope.row.state === 1">删除</el-button>
                </template>
              </el-table-column>

            </el-table>
              <!-- 分页 -->
              <el-pagination
                v-if="total > this.pagesize"
                background
                style="margin-top:20px;text-align:right"
                layout="prev, pager, next, sizes, jumper"
                :total="total"
                :page-size="pagesize"
                :current-page="page"
                :page-sizes="[5,10,20,50]"
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
              ></el-pagination>
          </el-col>
        </el-row>
      </el-card>
    </div>
  </div>
</template>

<script>
import { list, changeState } from '@/api/hmmm/articles'

export default {
  name: 'ArticlesInterviewing',
  data () {
    return {
      // 顶部表单
      form: {
        keyword: '',
        state: null
      },
      // 第几页
      page: 1,
      // 一页多少
      pagesize: 10,
      // 总条数
      total: 0,
      // 下边表单
      ArticlesList: []
    }
  },
  // 页面加载时调用
  created () {
    this.getAtricleList()
  },
  methods: {
    // 清空表单
    handleClear () {
      this.form = {
        keyword: '',
        state: null
      }
    },
    // 查询列表
    async getAtricleList () {
      const { data: { items, counts } } = await list({
        ...this.form,
        page: this.page,
        pagesize: this.pagesize
      })
      // 将获取到的数据赋值给ArticlesList
      // console.log(items)
      this.ArticlesList = items
      this.total = counts
    },
    // currentPage 改变时会触发 当每页显示条数发生变化
    handleCurrentChange (val) {
      // console.log(val)
      this.page = val
      this.getAtricleList()
    },
    // 序号递增 下一页时累加
    indexMethod (index) {
      return (index + 1) + this.pagesize * (this.page - 1)
    },
    // pageSize 改变时会触发  当前页发生变化
    handleSizeChange (size) {
      // console.log(size, this.total)
      this.pagesize = size
      this.getAtricleList()
    },
    async handleArticlesState (str) {
      // console.log(str.state)
      await changeState({
        id: str.id,
        state: str.state
      }
      )
      // 判断str.state是否是启用状态,如果是则返回true否则为false 并重新赋值给str.state
      str.state = str.state === 1 ? 0 : 1
      console.log(str.state)
    }

  }
}
</script>

<style lang="scss" scoped>
// 顶部表单样式
.demo-form-inline {
  .key-word {
    margin-left: 30px;
  }
  .state {
    margin-left: 30px;
  }
}
</style>
