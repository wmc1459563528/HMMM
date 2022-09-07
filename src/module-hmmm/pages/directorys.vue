<template>
  <div class="dashboard-container">
    <div class="app-container">
      <el-card shadow="never">
        目录管理
      </el-card>
    </div>
  </div>
</template>

<script>
import { list as getCompanys } from '@/api/hmmm/companys'
import { list as getQuestion } from '@/api/hmmm/questions'

export default {
  created () {
    getCompanys()
    getQuestion()
  }
}
</script>

<style scoped></style>
