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
              <el-button icon="el-icon-edit" type="success">新增目录</el-button>
            </el-form-item>
          </el-form>
        </el-col>
      </el-card>
    </div>
    <tags-add ref="tagssAdd"></tags-add>
  </div>
</template>

<script>
import { list } from '@/api/hmmm/tags'
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
  methods: {
    // 清除
    clear () {
      this.requestParams = {
        tagsName: null,
        state: null,
        page: 1,
        pagesize: 10
      }
    },
    // 筛选
    filter () {
      this.requestParams.page = 1
      this.getList()
    },
    // 获取列表
    async getList () {
      this.tagPage.subjectID = this.subject.id || null
      const { data } = await list(this.tagPage)
      this.tags = data.items
      this.total = data.counts
    }
  }
}
</script>

<style lang="scss" scoped>
.tags-container{
  padding: 10px;
}
</style>
