<template>
  <div class="container">
    <!-- 编辑/新增对话框 -->
    <el-dialog
      title="新增文章"
      :visible="isShow"
      @close="handleClose"
    >
      <el-form
        ref="form"
        :model="data"
        label-width="80px"
      >
        <el-form-item label="文章标题">
          <el-input v-model="data.title"></el-input>
        </el-form-item>
        <el-form-item label="文章内容">
          <!-- 富文本编辑器 -->
          <VueUeditorWrap />
        </el-form-item>
        <el-form-item label="视频地址">
            <el-input v-model="data.videoURL"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="handleClose">取消</el-button>
        <el-button type="primary" @click="handleSubmit">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
// 富文本编辑器
import VueUeditorWrap from 'vue-ueditor-wrap' // ES6 Module
export default {
  name: 'ArticlesAdd',
  props: {
    isShow: {
      type: Boolean,
      default: false
    },
    data: {
      type: Object,
      default: () => {}
    }

  },
  components: {
    VueUeditorWrap
  },
  methods: {
    handleClose () {
      this.$emit('update:isShow', false)
      // console.log(this.data)
    },
    handleSubmit () {

    }
  }
}
</script>

<style scoped lang="less">

</style>
