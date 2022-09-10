<template>
  <div class='container'>
    <!-- 预览对话框 -->
    <el-dialog
    title="预览文章"
    :visible="isShow"
    @close="handleClose"
  >
  <!-- 标题 -->
    <div class="title">
      <h2>{{ ArticlesList.title }}</h2>
    </div>
    <!-- 描述信息 -->
    <div class="message">
      <span>{{ ArticlesList.createTime | parseTimeByString }}</span>
      <span>{{ ArticlesList.username }}</span>
      <span class="el-icon-view"></span>
      <span>{{ ArticlesList.state }}</span>
    </div>
    <!-- 内容 -->
    <div class="content" v-html="ArticlesList.articleBody" />

  </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'ArticlesPreview',
  props: {
    // dialog的状态
    isShow: {
      type: Boolean,
      default: false
    },
    ArticlesList: {
      type: Object,
      default: () => {}
    }
  },
  methods: {
    // 关闭弹窗
    handleClose () {
      this.$emit('update:isShow', false)
    }
  }
}
</script>

<style lang="scss" scoped>
.container{
  width: 800px;
   .message{
    padding-left: 10px;
    span{
      margin-left:8px;
    }
    .el-icon-view{
      // 文字与图片中线对齐
      vertical-align:middle;
    }
  }
  .content{
    margin-top: 5px;
    padding-left: 10px ;
    border-top: 1px dashed  gray;
    background-color: #f5f5f5;
  }
}

</style>
