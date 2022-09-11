<template>
  <div class="container">
    <el-card class="box-card">
      <div slot="header" class="clearfix">
        <!-- 修改试题 -->
        <span>{{ $route.query.id ? "试题修改" : "试题录入" }}</span>
      </div>
      <div>
        <el-form
          ref="form"
          label-width="120px"
          :model="formData"
          :rules="rules"
        >
          <!-- 学科 -->
          <el-form-item label="学科：" prop="subjectID">
            <el-select
              placeholder="请选择"
              v-model="formData.subjectID"
              @change="subjectIDChange"
            >
              <el-option
                v-for="item in subjectList"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <!-- 目录 -->
          <el-form-item label="目录：" prop="catalogID">
            <el-select placeholder="请选择" v-model="formData.catalogID">
              <el-option
                v-for="item in catalogList"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <!-- 企业 -->
          <el-form-item label="企业：" prop="enterpriseID">
            <el-select placeholder="请选择" v-model="formData.enterpriseID">
              <el-option
                v-for="item in enterpriseIDLists"
                :key="item.id"
                :label="item.company"
                :value="item.id"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <!-- 城市 -->
          <el-form-item label="城市：" class="city-select" prop="city">
            <el-select
              placeholder="请选择"
              v-model="formData.province"
              @change="cityChange"
            >
              <el-option
                v-for="(item, index) in provinces"
                :key="index"
                :label="item"
                :value="item"
              >
              </el-option>
            </el-select>
            <!-- city -->
            <el-select placeholder="请选择" v-model="formData.city">
              <el-option
                v-for="(item, index) in citys"
                :key="index"
                :label="item"
                :value="item"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <!-- 方向 -->
          <el-form-item label="方向：" prop="direction">
            <el-select placeholder="请选择" v-model="formData.direction">
              <el-option
                v-for="(item, index) in direction"
                :key="index"
                :label="item"
                :value="item"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <!-- 题型 -->
          <el-form-item label="题型：" prop="questionType">
            <el-radio-group v-model="formData.questionType">
              <el-radio
                v-for="item in questionType"
                :key="item.value"
                :label="item.value + ''"
                >{{ item.label }}</el-radio
              >
            </el-radio-group>
          </el-form-item>
          <!-- 难度 -->
          <el-form-item label="难度：" prop="difficulty">
            <el-radio-group v-model="formData.difficulty">
              <el-radio
                v-for="item in difficulty"
                :key="item.value"
                :label="item.value + ''"
                >{{ item.label }}</el-radio
              >
            </el-radio-group>
          </el-form-item>

          <!-- 题干 -->
          <el-form-item label="题干：" prop="question">
            <quill-editor
              ref="quillEditor"
              @blur="checkQustion"
              v-model="formData.question"
              :options="editorOptions"
              style="height: 150px; margin-bottom: 70px"
            ></quill-editor>
          </el-form-item>
          <!-- 选项 -->
          <el-form-item label="选项：" v-if="formData.questionType !== '3'">
            <div v-if="formData.questionType === '1'">
              <el-row
                style="margin-bottom: 20px"
                type="flex"
                v-for="(item, index) in formData.options"
                :key="item.code"
              >
                <!-- 数据进行转换join数据 ----------------->
                <el-radio
                  :label="true"
                  v-model="item.isRight"
                  @change="optionsChange(item)"
                  style="margin-right: 9px"
                  >{{ item.code + ":" }}</el-radio
                >
                <el-input
                  style="width: 250px"
                  v-model="formData.options[index].title"
                ></el-input>
                <UplodaImage></UplodaImage>
              </el-row>
            </div>
            <!-- </el-radio-group> -->
            <!-- 多选 -->
            <div v-if="formData.questionType === '2'">
              <el-row
                style="margin-bottom: 20px"
                type="flex"
                justify="start"
                align="middle"
                v-for="(item, index) in formData.options"
                :key="item.code"
              >
                <el-checkbox
                  :label="true"
                  v-model="item.isRight"
                  style="margin-right: 9px"
                  >{{ item.code + ":" }}</el-checkbox
                >

                <el-input
                  style="width: 250px"
                  v-model="formData.options[index].title"
                ></el-input>
                <UplodaImage></UplodaImage>
              </el-row>
            </div>

            <el-row
              ><el-button
                type="danger"
                size="medium"
                :disabled="formData.questionType !== '2'"
                @click="addOption"
                ><i class="el-icon-plus"></i>增加选项与答案</el-button
              ></el-row
            >
          </el-form-item>
          <!-- 解析视频 -->
          <el-form-item label="解析视频：">
            <el-input
              style="width: 400px"
              v-model="formData.videoURL"
            ></el-input>
          </el-form-item>
          <!-- 答案解析 -->
          <el-form-item label="答案解析:" prop="answer">
            <quill-editor
              ref="quillEditor"
              @blur="checkQustion"
              v-model="formData.answer"
              :options="editorOptions"
              style="height: 150px; margin-bottom: 70px"
            ></quill-editor>
          </el-form-item>
          <!-- 题目备注 -->
          <el-form-item label="题目备注：">
            <el-input
              style="width: 400px"
              type="textarea"
              rows="4"
              v-model="formData.remarks"
            ></el-input>
          </el-form-item>
          <!-- 试题标签 -->
          <el-form-item label="试题标签：">
            <el-select
              placeholder="请选择"
              multiple
              filterable
              allow-create
              default-first-option
              v-model="formData.tags"
            >
              <el-option
                v-for="item in tagsLists"
                :key="item.value"
                :label="item.label"
                :value="item.label"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <!-- 提交按钮 -->
          <el-form-item>
            <el-button v-if="!$route.query.id" @click="submit()" type="primary"
              >确认提交</el-button
            >
            <el-button v-else @click="update()" type="success"
              >确认修改</el-button
            >
          </el-form-item>
        </el-form>
      </div>
    </el-card>
  </div>
</template>

<script>
import { add, detail, update } from '@/api/hmmm/questions'
import { provinces, citys } from '@/api/hmmm/citys'
import { direction, difficulty, questionType } from '@/api/hmmm/constants'
import { simple } from '@/api/hmmm/subjects'
import { simple as catalog } from '@/api/hmmm/directorys'
import { list as enterpriseIDList } from '@/api/hmmm/companys'
import { simple as tagsList } from '@/api/hmmm/tags'
import { quillEditor } from 'vue-quill-editor'
import { getToken } from '@/utils/auth'
import 'highlight.js/styles/vs2015.css'
import 'quill/dist/quill.core.css'
import 'quill/dist/quill.snow.css'
import 'quill/dist/quill.bubble.css'
import UplodaImage from '../components/uploda-image.vue'

export default {
  name: 'QuestionsNew',
  components: {
    quillEditor,
    UplodaImage
  },
  data () {
    return {
      enterpriseIDLists: '',
      tagsLists: '',
      token: getToken(),
      selectCode: true,

      // 4.富文本输入的时候高亮效果
      editorOptions: {
        placeholder: '请输入0-500位字符',
        modules: {
          // id设置
          toolbar: [
            ['bold', 'italic', 'underline', 'strike'],
            [{ list: 'ordered' }, { list: 'bullet' }],
            ['blockquote'],
            ['code-block', 'image', 'link']
          ]
        }
      },
      // 表单校验
      rules: {
        subjectID: [
          {
            required: true,
            message: '学科内容不能为空',
            trigger: ['change', 'blue']
          }
        ],
        catalogID: [
          {
            required: true,
            message: '目录内容不能为空',
            trigger: ['change', 'blue']
          }
        ],
        enterpriseID: [
          {
            required: true,
            message: '企业内容不能为空',
            trigger: ['change', 'blue']
          }
        ],
        city: [
          {
            required: true,
            message: '地区内容不能为空',
            trigger: ['change', 'blue']
          }
        ],
        direction: [
          {
            required: true,
            message: '方向内容不能为空',
            trigger: ['change', 'blue']
          }
        ],
        questionType: [
          {
            required: true,
            message: '题型内容不能为空',
            trigger: ['change', 'blue']
          }
        ],
        difficulty: [
          {
            required: true,
            message: '难度内容不能为空',
            trigger: ['change', 'blue']
          }
        ],
        question: [
          {
            required: true,
            message: '请输入题干',
            trigger: ['change', 'blue']
          }
        ],
        answer: [
          {
            required: true,
            message: '请输入答案解析',
            trigger: ['change', 'blue']
          }
        ]
      },
      formData: {
        // 学科
        subjectID: null,
        catalogID: null,
        // 目录
        enterpriseID: null,
        // 企业
        province: null,
        // 城市-地区
        city: null,
        // 地区
        direction: null,
        // 方向
        questionType: '1',
        // 题型
        difficulty: '1',
        // 难度
        question: null,
        // 企业渲染
        enterpriseIDLists: [],
        tagsLists: [],
        // 题干进行回显
        options: [
          {
            code: 'A',
            title: '',
            img: '',
            isRight: false
          },
          {
            code: 'B',
            title: '',
            img: '',
            isRight: false
          },
          {
            code: 'C',
            title: '',
            img: '',
            isRight: false
          },
          {
            code: 'D',
            title: '',
            img: '',
            isRight: false
          }
        ],
        videoURL: '',
        answer: '',
        remarks: ''
      },
      // 数据渲染
      difficulty,
      questionType,
      // 城市下拉菜单
      provinces: [],
      // 城市地区
      citys: [],
      direction: [],
      // 学科渲染
      subjectList: [],
      // 目录渲染
      catalogList: []
    }
  },
  // 初始化
  created () {
    this.provinces = provinces()
    this.direction = direction
    this.simple()
    this.enterpriseIDList()
    this.tagsList()
    if (this.$route.query.id) {
      this.gitDetail()
    }
  },
  methods: {
    // 富文本编辑器
    catchData1 (item) {
      this.formData.question = item
    },
    catchData2 (item) {
      this.formData.answer = item
    },
    optionsChange (item) {
      // 1. 清除所有的选中
      this.formData.options.forEach((item) => {
        item.isRight = false
      })
      // 2. 自己选中
      item.isRight = true
    },
    cityChange () {
      this.formData.city = null
      this.citys = citys(this.formData.province)
    },
    async subjectIDChange (subjectID) {
      this.formData.catalogID = null
      const res1 = await catalog(this.formData.subjectID)
      this.catalogList = res1.data
      this.formData.tags = null
      const res2 = await tagsList(this.formData.subjectID)
      this.tagsLists = res2.data
      console.log(this.tagsLists)
    },
    async simple () {
      const { data } = await simple()
      this.subjectList = data
    },
    async enterpriseIDList () {
      const {
        data: { items }
      } = await enterpriseIDList({ pagesize: 10000 })
      this.enterpriseIDLists = items
      console.log(items)
      console.log(this.enterpriseIDLists)
    },
    async tagsList () {
      const { data } = await tagsList()
      this.tagsLists = data
    },
    addOption () {
      // 点击新增选项
      this.formData.options.push({
        code: String.fromCharCode(65 + this.formData.options.length),
        title: '',
        img: '',
        isRight: false
      })
    },
    checkQustion () {
      // 验证富文本长度
      this.$refs.form.validateField('question')
      //  console.log(this.$refs.quillEditor.value.length)
      if (this.$refs.quillEditor.value.length >= 500) {
        this.$message.warning('题干或答案解析的内容长度不能超过500个字符')
      }
    },
    async gitDetail () {
      const { data } = await detail({ id: this.$route.query.id })
      console.log(data, data.tags)
      if (data.tags) data.tags = data.tags.split(',')
      data.options = data.options.map((item) => {
        item.isRight = item.isRight === 1
        return item
      })
      this.formData = data
      const res = await catalog(this.formData.subjectID)
      this.catalogList = res.data
      const res2 = await tagsList(this.formData.subjectID)
      this.tagsLists = res2.data
    },
    // 提交
    submit () {
      // console.log(this.formData, 1111)
      this.$refs.form.validate(async flag => {
        if (!flag) return
        const { data: res } = await add(this.formData)
        console.log(res)
        this.$message.success('试题录入成功')
        this.$router.push('/questions/list')
        // console.log(1)
      })
    },
    // 修改
    update () {
      this.$refs.form.validate(async (flag) => {
        if (!flag) return
        const res = await update(this.formData)
        console.log(res)
        this.$message.success('修改成功')
        this.$router.push('/questions/list')
        // console.log(1)
      })
    }
  }
}
</script>

<style scoped lang="scss">
.container {
  padding: 20px;
}
.text {
  font-size: 14px;
}

.item {
  margin-bottom: 20px;
}

.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}
.clearfix:after {
  clear: both;
}

.el-select {
  width: 400px;
}
.city-select {
  .el-select {
    padding-right: 5px;
    width: 203px;
  }
}

.fuwenben {
  ::v-deep.w-e-toolbar {
    z-index: 0 !important;
  }
  ::v-deep.w-e-text-container {
    border: 1px solid white;
    border-top: none;
    height: 200px;
    z-index: 0;
  }
}
.avatar-uploader {
  ::v-deep .el-upload--picture-card {
    width: 100px;
    height: 60px;
    margin-left: 4px;
    z-index: 1;
    position: relative;
    span {
      display: block;
      width: 100px;
      height: 60px;
      line-height: 60px;
      color: black;
      font-size: 15px;
    }
    i {
      font-size: 20px;
      position: absolute;
      right: -10px;
      top: -9px;
    }
  }
}
</style>
