<template>
  <div class="add-form">
    <el-form
      :rules="ruleInline"
      ref="dataForm"
      :model="formData"
      label-position="left"
      label-width="150px"
      style="width: 80%; margin-left:10px;"
    >
      <el-form-item label="企业名称" prop="shortName">
        <el-input v-model="formData.shortName"></el-input>
        <el-checkbox v-model="formData.isFamous">是否为名企</el-checkbox>
      </el-form-item>
      <el-form-item label="所属公司" prop="company">
        <el-input v-model="formData.company"></el-input>
        <p>https://www.tianyancha.com （在此可查询所属公司全称及简称）</p>
      </el-form-item>
      <el-form-item label="城市">
        <MyCity v-model="formData.provAndCity"/>
      </el-form-item>
      <el-form-item label="方向（企业标签）" prop="tags">
        <el-input v-model="formData.tags"></el-input>
      </el-form-item>
      <el-form-item label="备注" prop="remarks">
        <el-input
          type="textarea"
          :autosize="{ minRows: 2, maxRows: 4}"
          placeholder="请输入"
          v-model="formData.remarks"
        ></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="hAddorUpdate">{{$t('table.confirm')}}</el-button>
        <el-button v-if="cIsShowCancel" @click="$emit('close')">{{$t('table.cancel')}}</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>
<script>
// import { update, add } from '@/api/base/users'
import { detail, add, update } from '@/api/hmmm/companys'
import MyCity from './citys'
export default {
  name: 'CompanysAdd',
  components: {
    MyCity
  },
  props: {
    // 接收从复组件中传入的公司编号
    companyId: {
      type: [String, Number], // 这个属性可以是字符串，也可以是Number
      required: false // 这个属性并不是必须的
    }
  },
  data () {
    return {
      formData: {
        isFamous: false, // 是否是名企
        shortName: '', // 简写
        company: '', // 名字
        provAndCity: { // 城市
          province: '',
          city: ''
        },
        tags: '', // 标签
        remarks: '' // 备注
      },
      // 表单验证
      ruleInline: {
        shortName: [
          { required: true, message: '企业简称不能为空', trigger: 'blur' }
        ],
        tags: [{ required: true, message: '请请输标签', trigger: 'blur' }]
      }
    }
  },
  // 创建完毕状态
  created () {
    // 是否传入了公司编号。
    // 如果传入，说明是编辑，否则是添加
    if (this.companyId) {
      this.loadDetail()
    }
  },
  methods: {
    // 在编辑情况下，获取详情
    async loadDetail () {
      const { companyId } = this
      if (!companyId) {
        return
      }
      this.formData.id = companyId
      const { data: res } = await detail({ id: companyId })
      // 获取详情 给formData
      res.provAndCity = {
        province: res.province,
        city: res.city
      }
      res.isFamous = !!res.isFamous
      this.formData = res
    },
    // 做编辑
    async doUpdate () {
      const data = {
        ...this.formData,
        province: this.formData.provAndCity.province,
        city: this.formData.provAndCity.city
      }
      await update(data)
      this.emit()
    },
    // 做添加
    async doAdd () {
      try {
        await add({
          ...this.formData,
          province: this.formData.provAndCity.province,
          city: this.formData.provAndCity.city
        })
        this.emit()
      } catch (err) {
        console.log(err)
      }
    },
    // 告诉父组件完成了
    emit () {
      // this.$listeners: 自动会收集所有加在这个组件上的事件监听
      this.$listeners.close && this.$emit('close')
      this.$listeners.refresh && this.$emit('refresh')
    },
    // 表单提交
    hAddorUpdate () {
      this.$refs.dataForm.validate(valid => {
        // 趁早返回
        if (!valid) {
          this.$message.error('*号为必填项!')
          return
        }
        // 如果有companyId说明是编辑，做doUpdate
        this.companyId ? this.doUpdate() : this.doAdd()
      })
    }
  },
  computed: {
    cIsShowCancel () {
      return !!this.$listeners.close
    }
  }
}
</script>
<style>
.el-form--label-left .el-form-item__label {
  text-align: right;
}
.el-form-item--medium {
  margin-bottom: 22px;
}
.el-dialog__footer {
  text-align: center;
}
</style>
