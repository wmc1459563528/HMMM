<template>
  <div class="articles-container">
    <el-card>
      <!-- 顶部表单部分 -->
      <el-row>
        <el-col :span="22">
          <el-form label-width="80px" size="small" :inline="true">
            <el-form-item label="关键字">
              <el-input style="200px" v-model="form.keyword"></el-input>
           </el-form-item>
            <el-form-item label="状态">
              <el-select v-model="form.state">
                <el-option :value="1" label="启用"></el-option>
                <el-option :value="0" label="禁用"></el-option>
              </el-select>
            </el-form-item>
            <el-form-item style="text-align:right">
              <el-button @click="handleClear()">清除</el-button>
              <el-button @click="handleSearch()" type="primary">搜索</el-button>
            </el-form-item>
          </el-form>
        </el-col>
        <el-col :span="2" style="text-align:right">
          <el-button icon="el-icon-edit" @click="handleOpenDialog()" type="success" size="small">新增技巧</el-button>
        </el-col>
      </el-row>
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
      <!-- 表格 -->
    <el-row v-loading="loading"  element-loading-text="拼命加载中">
      <el-col :span="24">
       <el-table :data="ArticlesList" style="width: 100%">
              <el-table-column
                type="index"
                label="序号"
                :index="indexMethod"
                width="80"
              />
          <el-table-column label="文章标题" width="400px">
            <template slot-scope="scope">
              {{scope.row.title}}
               <!-- href="javascript:;"阻止a链接的默认行为 也可以给click绑定prevent  -->
              <a @click="handleVideo(scope.row.videoURL)" v-if="scope.row.videoURL" href="javascript:;" class="el-icon-film"></a>
            </template>
          </el-table-column>
          <el-table-column label="阅读数" prop="visits"></el-table-column>
          <el-table-column label="录入人" prop="username"></el-table-column>
          <el-table-column label="录入时间">
            <!--
              如果和父组件中先用到子组件中的数据 就需要用到 slot-scope
              scope可以获取到 row, column, $index 和 store（table 内部的状态管理）的数据
             -->
            <!-- 利用管道符来修改时间格式 -->
            <template slot-scope="scope">{{scope.row.createTime|parseTimeByString}}</template>
          </el-table-column>
          <el-table-column label="状态" width="100px">
            <template slot-scope="scope">{{scope.row.state===1?'已启用':'已禁用'}}</template>
          </el-table-column>
          <el-table-column label="操作" width="180px">
            <template slot-scope="scope">
              <el-button type="text" @click="handlePreview(scope.row)">预览</el-button>
              <el-button type="text" @click="handletoggleState(scope.row)">{{scope.row.state===1?'禁用':'启用'}}</el-button>
              <el-button type="text" @click="handleOpenDialog(scope.row)" :disabled="scope.row.state===1">修改</el-button>
              <el-button type="text" @click="handleDelArticle(scope.row)" :disabled="scope.row.state===1||scope.row.totals > 0">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-col>
    </el-row>
      <!-- 分页 -->
      <el-pagination
        style="margin-top:20px;text-align:right"
        background
        layout="prev, pager, next, sizes, jumper"
        :total="total"
        :page-size="pagesize"
        :current-page="page"
        @current-change="handleCurrentChange"
        :page-sizes="[5,10,20,50]"
        @size-change="handleSizeChange"
      ></el-pagination>
    </el-card>
    <!-- 添加/修改 -->
    <articles-add ref="articlesAdd" :data="currArticle" @updateList="handelUpdateList()" />
        <!-- 预览框 -->
      <articles-preview
        :articles-list="currArticle"
        :isShow.sync="isShowDialogVisible"
      />
      <!-- 音频 -->
    <div v-if="videoURL">
        <div @click="handleClose" class="el-icon-close">
          <video ref="video" :src="videoURL" controls></video>
        </div>
    </div>
  </div>
</template>

<script>
import { list, changeState, remove } from '@/api/hmmm/articles'
import ArticlesAdd from '../components/articles-add'
import ArticlesPreview from '../components/articles-preview'
export default {
  name: 'ArticlesList',
  components: { ArticlesAdd, ArticlesPreview },
  data () {
    return {
      form: {
        keyword: '',
        state: null
      },
      page: 1,
      pagesize: 10,
      total: 0,
      ArticlesList: [],
      currArticle: {},
      videoURL: null,
      // 预览状态
      isShowDialogVisible: false,
      loading: false
    }
  },
  created () {
    this.getAtricleList()
  },
  methods: {
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
    // 序号递增 下一页时累加
    indexMethod (index) {
      return index + 1 + this.pagesize * (this.page - 1)
    },
    // 切换分页
    handleCurrentChange (val) {
      this.page = val
      this.getAtricleList()
    },
    // pageSize 改变时会触发  当前页发生变化
    handleSizeChange (size) {
      this.page = 1
      this.pagesize = size
      this.getAtricleList()
    },

    // 切换状态
    async handletoggleState (article) {
      await changeState({
        id: article.id,
        state: article.state === 1 ? 0 : 1
      })
      // 提示用户
      this.$message.success('操作成功')
      // 判断str.state是否是启用状态,如果是则返回true否则为false 并重新赋值给str.state
      article.state = article.state === 1 ? 0 : 1
    },
    // 视频
    handleVideo (url) {
      this.videoURL = url
      /*
     因为this.$refs.video外层用了v-if,所以获取到的this.$refs.video值是undefined
     用nextTick让它在微任务时候执行
      */
      this.$nextTick(() => {
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
    // 清除筛选条件
    handleClear () {
      this.form = {
        keyword: '',
        state: null
      }
      this.getAtricleList()
    },
    // 进行筛选
    handleSearch () {
      this.page = 1
      this.getAtricleList()
    },

    // 预览
    // 文章预览
    handlePreview (val) {
      this.isShowDialogVisible = true
      this.currArticle = val
      // console.log(val)
    },
    // 添加&&编辑
    handleOpenDialog (article = {}) {
      this.currArticle = article
      this.$nextTick(() => {
        this.$refs.articlesAdd.open()
      })
    },

    // 删除
    async handleDelArticle (article) {
      await this.$confirm('此操作将永久删除该文章, 是否继续?', '提示', { type: 'warning' })
      await remove(article)
      this.$message.success('删除成功')
      this.getAtricleList()
    },

    // 新增&&修改 后更新列表
    handelUpdateList () {
      if (!this.currArticle.id) {
        this.page = 1
      }
      this.getAtricleList()
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
