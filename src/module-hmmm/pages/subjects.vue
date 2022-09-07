<template>
  <div class="dashboard-container">
    <div class="app-container">
      <el-card shadow="never">
        学科管理
        <el-form>
          <h3>{{form.name}}</h3><el-form-item>
            <el-select v-model="form.province">
              <el-option v-for="item in list" :key="item" :label="item" :value="item"></el-option>
              </el-select>
              </el-form-item>
        </el-form>
      </el-card>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {

      list: [], // 从接口处取回
      form: {
        name: 'abc',
        province: '潜江' // 从接口处取回
      }
    }
  },
  created () {
    setTimeout(() => {
      // ajax
      const res = {
        name: 'abc',
        list: ['潜江', '滨海', '北京'],
        province: '北京'
      }

      this.list = res.list
      this.form.name = res.name
      this.form = { name: res.name }
      // this.$set(this.form, 'province', res.province)
      this.form.province = res.province
    })
  }
}
</script>

<style scoped></style>
