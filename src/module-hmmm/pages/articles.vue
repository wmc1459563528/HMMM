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
                  <el-button size="small" type="primary" @click="handleSearch">搜索</el-button>
                </el-form-item>
              </el-form>
            </div>
          </el-col>
          <el-col :span="2">
            <div class="grid-content bg-purple">
              <el-button
              size="small"
              type="success"
              icon="el-icon-edit"
              @click="handelAddOrEdit()"
              >新增技巧</el-button>
            </div>
          </el-col>
        </el-row>
        <!-- 数据提示 -->
        <el-row>
          <el-col :span="24">
            <!-- 利用模板字符串来显示信息 :closable='false'不可关闭 -->
            <el-alert
              :title="`数据一共${total}条`"
              type="info"
              show-icon
              :closable="false"
            ></el-alert>
          </el-col>
        </el-row>
        <!-- 文章渲染 -->
        <el-row v-loading="loading"  element-loading-text="拼命加载中">
          <el-col :span="24">
            <el-table :data="ArticlesList" style="width: 100%">
              <el-table-column
                type="index"
                label="序号"
                :index="indexMethod"
                width="80"
              />
              <el-table-column label="文章标题" width="350">
                <template slot-scope="scope">
                  {{ scope.row.title }}
                  <!-- href="javascript:;"阻止a链接的默认行为 也可以给click绑定prevent  -->
                  <a
                    @click="handleVideo(scope.row)"
                    href="javascript:;"
                    class="el-icon-film"
                    v-if="scope.row.videoURL"
                  />
                </template>
              </el-table-column>
              <el-table-column prop="visits" label="阅读数" width="180" />
              <el-table-column prop="username" label="录入人" width="180" />
              <el-table-column label="录入时间" width="250">
                <!--
                  如果和父组件中先用到子组件中的数据 就需要用到 slot-scope
                  scope可以获取到 row, column, $index 和 store（table 内部的状态管理）的数据
                   -->
                <template slot-scope="scope">
                  <!-- 利用管道符来修改时间格式 -->
                  {{ scope.row.createTime | parseTimeByString }}
                </template>
              </el-table-column>
              <el-table-column label="状态" width="100">
                <template slot-scope="scope">
                  {{ scope.row.state === 1 ? "已启用" : "已禁用" }}
                </template>
              </el-table-column>
              <el-table-column label="操作">
                <template  slot-scope="scope">
                  <el-button type="text" @click="handlePreview(scope.row)">预览</el-button>
                  <el-button
                    type="text"
                    @click="handleArticlesState(scope.row)"
                    >{{ scope.row.state === 1 ? "禁用" : "启用" }}</el-button
                  >
                  <el-button
                  type="text"
                  :disabled="scope.row.state === 1"
                  @click="handelAddOrEdit(scope.row)"
                    >修改</el-button
                  >
                  <el-button
                  type="text"
                  :disabled="scope.row.state === 1"
                   @click="handelDelArticles(scope.row)"
                    >删除</el-button
                  >
                </template>
              </el-table-column>
            </el-table>
            <!-- 分页 -->
            <el-pagination
              background
              style="margin-top: 20px; text-align: right"
              layout="prev, pager, next, sizes, jumper"
              :total="total"
              :page-size="pagesize"
              :current-page="page"
              :page-sizes="[5, 10, 20, 50]"
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
            ></el-pagination>
          </el-col>
        </el-row>
      </el-card>
      <!-- 音频 -->
      <div v-if="videoURL">
        <div @click="handleClose" class="el-icon-close">
          <video ref="video" :src="videoURL" controls></video>
        </div>
      </div>
      <!-- 预览框 -->
      <articles-preview
        :articles-list="ArticlesData"
        :isShow.sync="isShowDialogVisible"
      />
      <!-- 新增/编辑弹出框 -->
      <articles-add
        :isShow.sync="isShowAddOrEdit"
        :data="ArticlesData"
       />
    </div>
  </div>
</template>

<script>
import { list, changeState } from '@/api/hmmm/articles'
// 引入预览文章组件
import ArticlesPreview from '../components/articles-preview'
import ArticlesAdd from '../components/articles-add.vue'

export default {
  name: 'ArticlesInterviewing',
  components: {
    ArticlesPreview,
    ArticlesAdd
  },
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
      ArticlesList: [],
      // 音频
      videoURL: null,
      // 预览状态
      isShowDialogVisible: false,
      // 新增/修改文章状态
      isShowAddOrEdit: false,
      // 右侧按钮中文章列表
      ArticlesData: {}
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
      // 重新渲染
      this.getAtricleList()
    },
    // 查询列表
    async getAtricleList () {
      // 增强用户体验 数据回来之前出现个加载框
      this.loading = true
      const {
        data: { items, counts }
      } = await list({
        ...this.form,
        page: this.page,
        pagesize: this.pagesize
      })
      // 将获取到的数据赋值给ArticlesList和total
      // console.log(items)
      this.ArticlesList = items
      this.total = counts
      // 还原加载框
      this.loading = false
    },
    // currentPage 改变时会触发 当每页显示条数发生变化
    handleCurrentChange (val) {
      // console.log(val)
      this.page = val
      this.getAtricleList()
    },
    // 序号递增 下一页时累加
    indexMethod (index) {
      return index + 1 + this.pagesize * (this.page - 1)
    },
    // pageSize 改变时会触发  当前页发生变化
    handleSizeChange (size) {
      // console.log(size, this.total)
      this.pagesize = size
      this.getAtricleList()
    },
    // 操作的禁用与启用
    async handleArticlesState (str) {
      // console.log(str)
      await changeState({
        id: str.id,
        state: str.state
      })
      // 提示用户
      this.$message.success('操作成功')
      // 判断str.state是否是启用状态,如果是则返回true否则为false 并重新赋值给str.state
      str.state = str.state === 1 ? 0 : 1
      // console.log(str.state)
    },
    //  文章标题的按钮操作
    handleVideo (video) {
      // console.log(video.videoURL)
      this.videoURL = video.videoURL
      /*
     因为this.$refs.video外层用了v-if,所以获取到的this.$refs.video值是undefined
     用nextTick让它在微任务时候执行
      */
      this.$nextTick(() => {
        // console.log(this.$refs.video)
        // play()开始播放视频。
        this.$refs.video.play()
      })
    },
    //  关闭视频
    handleClose () {
      // todo 貌似有BUG 有时候点击关闭之后还会有声音
      this.$nextTick(() => {
        // pause()暂停当前播放的视频。
        this.$refs.video.pause()
      })

      this.videoURL = null
    },
    // 文章预览
    handlePreview (articles) {
      this.isShowDialogVisible = true
      this.ArticlesData = articles
      console.log(articles)
    },
    // 搜索
    handleSearch () {
      this.page = 1
      this.getAtricleList()
    },
    // 新增/编辑
    handelAddOrEdit (articles = {}) {
      this.isShowAddOrEdit = true
      this.ArticlesData = articles
      // console.log(articles)
    },
    // 删除
    handelDelArticles (row) {
      console.log(row)
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
// 视频的图标设置
.el-icon-film {
  color: blue;
  font-size: 18px;
  // 字体图标与文字居中显示
  vertical-align: middle;
}
//  video
.el-icon-close {
  position: absolute;
  top: -75px;
  right: 50%;
  width: 50px;
  height: 50px;
  font-size: 30px;
  text-align: center;
  padding: 10px;
  color: #fff;
  background-color: rgba(107, 107, 107, 0.5);
  border-radius: 50%;
  cursor: pointer;
  z-index: 9999;
  video{
    position:absolute;
    left: -350px;
    top: 30px;
    width: 800px;
    height: 500px;
  }
}
</style>
