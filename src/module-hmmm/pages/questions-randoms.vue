<template>
<div class="dashboard-container">
    <div class="app-container">
      <!-- 筛选 -->
      <el-card shadow="never">
        <!-- 左侧按钮 -->
        <el-col :span="18">
          <el-form :inline="true" :model="Qran" class="demo-form-inline">
            <el-form-item label="关键字">
              <el-input v-model="Qran.keyword"></el-input>
            </el-form-item>
          </el-form>
        </el-col>
        <!-- 右侧按钮 -->
        <el-col :span="6" style="text-align:right">
          <el-form>
            <el-form-item>
              <el-button @click="clear()">清除</el-button>
              <el-button @click="filter()" type="primary">搜索</el-button>
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
      <!-- 列表 -->
      <el-table :data="questions" v-loading="loading"  element-loading-text="拼命加载中">
        <el-table-column label="编号" prop="id" width="220px"></el-table-column>
        <el-table-column label="题型" width="80px">
          <template slot-scope="scope">
            {{questionType.find(item=>item.value===+scope.row.questionType).label}}
          </template>
        </el-table-column>
        <el-table-column label="题目编号" width="220px">
          <template slot-scope="scope">
            <div v-for="item in scope.row.questionIDs" :key="item.number">
              <a href="#" @click.prevent="openPreviewDialog(item)" style="color:#09f;">{{item.number}}</a>
            </div>
          </template>
        </el-table-column>
        <el-table-column label="录入时间" width="180">
          <template slot-scope="scope">
            {{scope.row.addTime|parseTimeByString}}
          </template>
        </el-table-column>
        <el-table-column label="答题时间(s)" prop="totalSeconds"></el-table-column>
        <el-table-column label="正确率(%)" prop="accuracyRate" ></el-table-column>
        <el-table-column label="录入人" prop="userName"></el-table-column>
        <el-table-column label="操作" width="80px">
          <template slot-scope="scope">
            <el-button
            plain type="danger"
            size="small"
            circle icon="el-icon-delete"
            @click="delQuestion(scope.row)"
            title="删除"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <el-pagination
        style="margin-top:20px;text-align:right"
        background
        layout="prev, pager, next, sizes, jumper"
        :total="total"
        :page-size="Qran.pagesize"
        :current-page="Qran.page"
        @current-change="pager"
        :page-sizes="[20,30,50,100]"
        @size-change="handleSizeChange"
      ></el-pagination>
      <!-- <el-pagination
        style="margin-top:20px;text-align:right"
        :page-size="20"
        :pager-count="11"
        layout="prev, pager, next"
        :total="1000">
      </el-pagination> -->
    </div>
    <!-- 预览 -->
    <questions-preview ref="questionPreview" :data="questionInfo"></questions-preview>
  </div>
</template>

<script>
// 引入难度和题型
import { difficulty, questionType } from '@/api/hmmm/constants'
// 引入题组列表和删除题组列表
import { randoms as questionList, removeRandoms as questionDel } from '@/api/hmmm/questions'
// 引入弹框
import QuestionsPreview from '../components/questions-preview'
export default {
  components: {
    QuestionsPreview
  },
  data () {
    return {
      questionInfo: {},
      questionType,
      difficulty,
      total: 0,
      questions: [],
      loading: false,
      Qran: {
        keyword: null,
        page: 1,
        pagesize: 20
      }
    }
  },
  // 预加载
  async created () {
    this.getList()
  },
  methods: {
    // 获取列表
    async getList () {
      this.loading = true
      const { data: questionsData } = await questionList(this.Qran)
      this.questions = questionsData.items
      this.total = questionsData.counts
      this.loading = false
    },
    // 输出题目编号
    openPreviewDialog (questionInfo) {
      this.questionInfo = questionInfo
      this.$nextTick(() => {
        this.$refs.questionPreview.open()
      })
    },
    // 清除
    clear () {
      this.Qran.keyword = null
      this.getList()
    },
    // 筛选
    filter () {
      this.Qran.page = 1
      this.getList()
    },
    // 删除问题
    async delQuestion (question) {
      await this.$confirm('此操作将永久删除该题组, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
      await questionDel({ id: question.id })
      // 如果为当前页的第一项，且数组长度为1则需要当前页-1
      if (this.questions.length === 1 && this.Qran.page > 1) this.Qran.page--
      // console.log(this.questions.length, this.Qran.page)
      this.$message.success('删除成功')
      this.getList()
    },
    // 换页
    handleSizeChange (size) {
      this.Qran.page = 1
      this.Qran.pagesize = size
      this.getList()
    },
    pager (np) {
      this.Qran.page = np
      this.getList()
    }
  }
}
</script>

<style scoped></style>
