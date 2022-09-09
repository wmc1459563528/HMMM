<template>
  <div class="articles-add">
    <el-dialog :title="`${data.id?'修改':'新增'}文章`" :visible.sync="show" width="800px">
      <el-form ref="form" size="small" label-width="100px" :model="formModel" :rules="formRules">
        <el-form-item label="文章标题：" prop="title">
          <el-input v-model="formModel.title" placeholder="请输入文章标签"></el-input>
        </el-form-item>
        <el-form-item label="文章内容：" prop="articleBody">
          <quill-editor v-model="formModel.articleBody" :options="editorOptions"></quill-editor>
        </el-form-item>
        <el-form-item label="视频地址：" prop="videoURL">
          <el-input v-model="formModel.videoURL" placeholder="请输入视频地址"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer">
        <el-button @click="show=false">取消</el-button>
        <el-button @click="confirm()" type="primary">确认</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { add, update } from '@/api/hmmm/articles'
import 'quill/dist/quill.core.css'
import 'quill/dist/quill.snow.css'
import 'quill/dist/quill.bubble.css'
import { quillEditor } from 'vue-quill-editor'
export default {
  name: 'ArticlesAdd',
  components: { quillEditor },
  props: {
    data: {
      default: () => {},
      type: Object
    }
  },
  data () {
    return {
      show: false,
      formModel: {
        id: null,
        title: null,
        articleBody: null,
        videoURL: null
      },
      formRules: {
        // 校验表单必填项
        title: [
          { required: true, message: '请输入文章标题', tirgger: ['blur', 'change'] }
        ],
        articleBody: [
          { required: true, message: '请输入文章内容', tirgger: ['blur', 'change'] }
        ]
      },
      editorOptions: {
        placeholder: '请输入正文',
        modules: {
          toolbar: [
            ['bold', 'italic', 'underline', 'strike'], // 加粗 斜体 下划线 删除线
            ['blockquote', 'code-block'], // 引用 代码块
            [{ list: 'ordered' }, { list: 'bullet' }], // 有序、无序列表

            [{ header: 1 }, { header: 2 }], // 1、2 级标题
            ['image', 'video', 'link'], // 链接、图片、视频、链接
            [{ indent: '-1' }, { indent: '+1' }], // 缩进
            ['clean'] // 清除文本格式
          ]

        }
      }
    }
  },
  methods: {
    open () {
      this.show = true
      if (this.data.id) {
        const { id, title, articleBody, videoURL } = this.data
        this.formModel = { id, title, articleBody, videoURL }
      } else {
        this.formModel = {
          id: null,
          title: null,
          articleBody: null,
          videoURL: null
        }
      }
      // 清除校验效果
      this.$nextTick(() => {
        this.$refs.form.clearValidate()
      })
    },
    async confirm () {
      this.$refs.form.validate(async valid => {
        if (valid) {
          // 提交
          if (!this.data.id) {
            // 新增
            await add(this.formModel)
          } else {
            // 修改
            this.formModel.id = this.data.id
            await update(this.formModel)
          }
          this.$message.success('操作成功')
          this.show = false
          this.$emit('updateList')
        }
      })
    }
  }
}
</script>
<style lang="scss" scoped>
.articles-add ::v-deep .ql-editor{
  height: 200px;
}
.articles-add ::v-deep .ql-toolbar.ql-snow{
  padding: 0 8px;
}
</style>
