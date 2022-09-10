<template>

  <div>

    <el-upload
      :class="{disabled: isDisabled}"
      style="margin-left: 4px"
      class="one"
      action=""

      list-type="picture-card"
      :file-list="fileList"
      :on-preview="handlePictureCardPreview"
      :limit="limit"
      :on-remove="handleRemove"
      :http-request="handleUpload"
      :on-change="handleChange"
      :before-upload="handleBeforeUpload"
    >

    </el-upload>
  </div>

</template>

<script>
import COS from 'cos-js-sdk-v5' // 导入腾讯云的包(sdk)

const cos = new COS({
  SecretId: 'AKIDfPzJOEo8D3cwc2CAvfFZdRzIKOErHRL0', // 身份识别ID
  SecretKey: 'VmnY3IpAF4dw3wZw3uVL4cvyoRpzxhDV' // 身份秘钥
})
export default {
  props: {
    // 图片上传的最大数量
    limit: {
      type: Number,
      default: 1
    }
  },
  data () {
    return {
      fileList: [

      ],
      //  图片预览对话框显示/隐藏
      dialogVisible: false,
      //  用于展示的图片地址,
      dialogImageUrl: ''

    }
  },
  computed: {
    // 控制上传按钮显示/隐藏
    isDisabled () {
      return this.fileList.length >= this.limit
    },
    // 该组件的所有图片是否都上传完毕(都上传完毕返回true, 只要存在没有传完的都返回false)
    isUploadAll () {
      return this.fileList.every(item => item.status === 'success')
    }
  },
  methods: {
    // 1. 图片预览
    handlePictureCardPreview (file) {
      // console.log(file)
      this.dialogImageUrl = file.url
      this.dialogVisible = true
    },
    // 删除
    handleRemove (file, fileList) {
      console.log(file, fileList, this.fileList)

      this.fileList = [...fileList]
    },
    // 上传
    handleUpload (obj) {
      // 0. 找到对应的图片文件
      const uid = obj.file.uid
      const file = this.fileList.find(item => item.uid === uid)

      file.status = 'uploading'
      // 使用腾讯云SDK上传图片文件到存储桶
      cos.putObject({
        Bucket: 'hrsass-1313596731', /* 存储桶, 必填字段 */
        Region: 'ap-shanghai', /* 存储桶所在区域, 必填字段 */
        Key: 'hrsass' + parseInt(Math.random() * 99999) + obj.file.name, /* 存储在桶里的对象键, 必填字段 */
        StorageClass: 'STANDARD',
        Body: obj.file, // 上传的文件对象
        onProgress: (progressData) => {
          file.percentage = parseInt(progressData.percent * 100)
        }
      }, (err, data) => {
        if (err) {
          this.$message.error('图片上传失败')
          return
        }
        // 1. 更新当前上传的对应图片的status为sucess
        // console.log(obj, data)

        setTimeout(() => {
          file.status = 'success'
        }, 1000)
        // 2. 把本地预览地址更新成图片服务器返回的url地址
        file.url = 'https://' + data.Location
      })
    },
    // 当文件操作时，同步本地文件数组
    handleChange (file, fileList) {
      this.fileList = [...fileList]
    },
    // 上传前图片文件校验
    handleBeforeUpload (file) {
      const isLt2M = file.size / 1024 / 1024 <= 2
      if (!isLt2M) {
        this.$message.warning('上传图片过大')
        return false
      }
      return true
    }
  }
}
</script>

<style lang="scss" scoped>

.disabled{
  ::v-deep{
    .el-upload--picture-card {
      display: none;
    }
  }
}

</style>
