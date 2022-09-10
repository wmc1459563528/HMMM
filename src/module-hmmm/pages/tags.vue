<template>
  <div class="dashboard-container">
    <div class="app-container">
      <!-- 面包屑 -->
      <div slot="header" v-if="subject.id && subject.name">
        <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item>学科管理</el-breadcrumb-item>
          <el-breadcrumb-item>{{subject.name}}</el-breadcrumb-item>
          <el-breadcrumb-item>目录管理</el-breadcrumb-item>
        </el-breadcrumb>
      </div>
      <!-- 筛选 -->
      <el-card shadow="never">
        <!-- 左侧按钮 -->
        <el-col :span="18">
          <el-form :inline="true" :model="tagPage" class="demo-form-inline">
            <el-form-item label="目录名称">
              <el-input v-model="tagPage.tagName"></el-input>
            </el-form-item>
            <el-form-item label="状态">
              <el-select v-model="tagPage.state" placeholder="请选择">
                <el-option label="启用" :value="1"></el-option>
                <el-option label="禁用" :value="0"></el-option>
              </el-select>
            </el-form-item>
            <el-form-item>
              <el-button @click="clear()">清除</el-button>
              <el-button type="primary" @click="filter()">搜索</el-button>
            </el-form-item>
          </el-form>
        </el-col>
        <!-- 右侧按钮 -->
        <el-col :span="6" style="text-align:right">
          <el-form>
            <el-form-item>
              <el-button v-if="subject.id && subject.name" type="text" icon="el-icon-back" @click="$router.back()">返回学科</el-button>
              <el-button icon="el-icon-edit" @click="addTag()" type="success">新增目录</el-button>
            </el-form-item>
          </el-form>
        </el-col>
      </el-card>
      <!-- 数据提示 -->
      <el-alert
        :title="`数据一共 ${total} 条`"
        style="margin-bottom:15px"
        type="info"
        class="alert"
        :closable="false"
        show-icon
      ></el-alert>
      <!-- 表格 -->
      <el-table :data="tags">
        <el-table-column label="序号" type="index" width="80px"></el-table-column>
        <el-table-column label="所属学科" prop="subjectName"></el-table-column>
        <el-table-column label="标签名称" prop="tagName"></el-table-column>
        <el-table-column label="创建者" prop="username"></el-table-column>
        <el-table-column label="创建日期">
          <template slot-scope="scope">{{scope.row.addDate|parseTimeByString}}</template>
        </el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">{{scope.row.state===1?'已启用':'已禁用'}}</template>
        </el-table-column>
        <el-table-column label="操作" width="150px">
          <template slot-scope="scope">
            <el-button type="text" @click="toggleState(scope.row)">{{scope.row.state===1?'禁用':'启用'}}</el-button>
            <el-button type="text" @click="addTag(scope.row)" :disabled="scope.row.state===1">修改</el-button>
            <el-button type="text" @click="delTag(scope.row)" :disabled="scope.row.state===1||scope.row.totals > 0">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <el-pagination
        style="margin-top:20px;text-align:right"
        background
        layout="prev, pager, next, sizes, jumper"
        :total="total"
        :page-size="tagPage.pagesize"
        :current-page="tagPage.page"
        @current-change="pager"
        :page-sizes="[5,10,20,50]"
        @size-change="handleSizeChange"
        hide-on-single-page
      ></el-pagination>
    </div>
    <!-- 添加和修改 -->
    <tags-add ref="tagssAdd" :data="currTag" @updateList="updateList()"></tags-add>
  </div>
</template>

<script>
import { list, changeState, remove } from '@/api/hmmm/tags'
import TagsAdd from '../components/tags-add'
export default {
  name: 'TagsList',
  components: { TagsAdd },
  data () {
    return {
      tagPage: {
        tagName: null,
        state: null,
        page: 1,
        pagesize: 10
      },
      total: 0,
      tags: [],
      currTag: {}
    }
  },
  computed: {
    subject () {
      return this.$route.query || {}
    }
  },
  created () {
    this.getList()
  },
  methods: {
    // 清除
    clear () {
      this.tagPage = {
        tagsName: null,
        state: null,
        page: 1,
        pagesize: 10
      }
    },
    // 筛选
    filter () {
      this.tagPage.page = 1
      this.getList()
    },
    // 获取列表
    async getList () {
      this.tagPage.subjectID = this.subject.id || null
      const { data } = await list(this.tagPage)
      this.tags = data.items
      this.total = data.counts
    },
    // 新增目录
    addTag (tag = {}) {
      this.currTag = tag
      this.$nextTick(() => {
        this.$refs.tagssAdd.open()
      })
    },
    // 添加/修改内容
    updateList () {
      if (!this.currTag.id) {
        this.tagPage.page = 1
      }
      this.getList()
    },
    // 禁用/启用按钮切换
    async toggleState (tag) {
      await changeState({
        id: tag.id,
        state: tag.state === 1 ? 0 : 1
      })
      this.$message.success('操作成功')
      tag.state = tag.state === 1 ? 0 : 1
    },
    // 删除
    async delTag (tag) {
      await this.$confirm('此操作将永久删除该标签, 是否继续?', '提示', {
        type: 'warning'
      })
      await remove(tag)
      this.$message.success('删除成功')
      this.getList()
    },
    // 切换条数
    handleSizeChange (size) {
      this.tagPage.page = 1
      this.tagPage.pagesize = size
      this.getList()
    },
    // 切换分页
    pager (np) {
      this.tagPage.page = np
      this.getList()
    }
  }
}
</script>

<style lang="scss" scoped>
.tags-container{
  padding: 10px;
}
</style>
