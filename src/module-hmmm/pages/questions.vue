<template>
  <div class="dashboard-container">
    <div class="app-container">
      <el-card shadow="never">
        <!-- 按钮 -->
        <div class="bth-wrapper">
      <el-button  type="success" icon="el-icon-edit" size="small">新增试题</el-button>
        </div>
        <!-- 筛选 -->
        <el-form label-width="80px" size="small">
        <el-row>
         <el-col :span="6">
            <el-form-item label="学科">
              <el-select v-model="requestParams.subjectID" placeholder="请选择">
              </el-select>
            </el-form-item>
          </el-col>
           <el-col :span="6">
            <el-form-item label="二级目录">
              <el-select v-model="requestParams.catalogID" style="width:100%">
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="标签">
              <el-select v-model="requestParams.tags" style="width:100%">
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="关键字">
              <el-input v-model="requestParams.keyword"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="试题类型">
              <el-select v-model="requestParams.questionType" style="width:100%">
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="难度">
              <el-select v-model="requestParams.difficulty" style="width:100%">
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="方向">
              <el-select v-model="requestParams.direction" style="width:100%">
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="录入人">
              <el-select v-model="requestParams.creatorID" style="width:100%">
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="题目备注">
              <el-input v-model="requestParams.remarks"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="企业简称">
              <el-input v-model="requestParams.shortName"></el-input>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item label="城市">
              <!-- 城市 -->
              <el-select @change="handleCity" v-model="requestParams.province" style="width:48%;margin-right:2%">
              </el-select>
              <!-- 地区 -->
              <el-select v-model="requestParams.city" style="width:50%">
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :span="6">
            <el-form-item style="text-align:right">
              <el-button>清除</el-button>
              <el-button type="primary">搜索</el-button>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
        <!-- 数据记录 -->
        <el-alert title="数据一共xx条" type="info" :closable="false" show-icon  style="margin-bottom:15px"></el-alert>
        <!-- 列表-->
      <el-table :data="questions">
        <el-table-column label="试题编号"></el-table-column>
        <el-table-column label="学科"></el-table-column>
        <el-table-column label="目录"></el-table-column>
        <el-table-column label="题型"></el-table-column>
        <el-table-column label="题干"></el-table-column>
        <el-table-column label="录入时间"></el-table-column>
        <el-table-column label="难度"></el-table-column>
        <el-table-column label="录入人"></el-table-column>
        <el-table-column label="操作"></el-table-column>
      </el-table>
        <!-- 分页 -->
      <el-pagination
       style="margin-top:20px;text-align:right"
       background
       layout="prev, pager, next,sizes,jumper"
       :page-sizes="[5,10,20,50]"
       :total="1000">
      </el-pagination>
      </el-card>
    </div>
  </div>
</template>

<script>
export default {
  nmae: 'question-page',
  data () {
    return {
      requestParams: {
        subjectID: null,
        difficulty: null,
        questionType: null,
        tags: null,
        province: null,
        city: null,
        keyword: null,
        remarks: null,
        shortName: null,
        direction: null,
        creatorID: null,
        catalogID: null,
        page: 1,
        pagesize: 5
      }
    }
  }
}
</script>

<style scoped lang='scss'>
.question-container {
  padding: 0 10px;
  margin: 10px 0;
}
.bth-wrapper{
    margin-bottom: 15px;
  text-align: right;
}

</style>
