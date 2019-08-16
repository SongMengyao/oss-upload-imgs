<template>
  <div class="upload-wrapper">
    <el-upload
      ref="upload"
      action="tokenData.host"
      :accept="type"
      :listType="listType"
      :limit="limit"
      :fileList="fileList"
      :before-upload="beforeUpload"
      :http-request="myUpload"
      :on-preview="handlePreview"
      :on-change="handleChange"
      :on-remove="handleRemove"
      :on-progress="handleProgress"
      :on-success="handleSuccess"
      :on-exceed="handleExceed"
    >
      <i class="el-icon-plus"></i>
    </el-upload>
    <el-dialog :visible.sync="previewVisible">
      <img width="100%" :src="previewImage" />
    </el-dialog>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  props: {
    type: {
      type: String,
      default: '.jpg, .jpeg, .png, .gif, .bmp, .JPG, .JPEG, .PBG, .GIF, .BMP'
    },
    listType: {
      type: String,
      default: 'picture-card'
    },
    limit: {
      type: Number,
      default: 1
    },
    defaultList: {
      type: String,
      default: ''
    }
  },
  data () {
    return {
      previewImage: '',
      previewVisible: false,
      tokenData: {
        host: ''
      },
      fileList: [],
      keyList: []
    }
  },
  mounted () {
    this.processUploadData()
  },
  methods: {
    // 添加默认图片
    processUploadData () {
      if (this.defaultList && this.defaultList !== '') {
        const urlArr = this.defaultList.split(',')
        urlArr.forEach(item => {
          const url = new URL(item)
          const pathname = url.pathname
          const keyArr = pathname.split('__')
          const filename = keyArr[1]
          const fileItem = {
            name: filename,
            url: url.href
          }
          this.fileList.push(fileItem)
          this.keyList.push(pathname.substring(1)) // 去掉pathname开头的 '/'
          this.$emit('onChange', this.keyList.join(','))
        })
      }
    },
    // 上传之前
    beforeUpload (file) {
      const filename = file.name
      if (filename.length < 120) {
        const params = {
          providerName: 'privateOSS',
          scope: 2
        }
        return this.$store.dispatch('getOssParam', params).then(res => {
          if (res) {
            this.tokenData = res
            this.param = {
              key: this.tokenData.dir + file.name + this.random_string(10),
              policy: this.tokenData.policy,
              OSSAccessKeyId: this.tokenData.accessKey,
              success_action_status: '200',
              signature: this.tokenData.signature
            }
          }
          return res
        })
      } else {
        this.$message.error('文件名称不能大于120')
        return false
      }
    },
    // 上传
    myUpload (content) {
      const { file } = content
      const param = new FormData()
      const key = `${this.tokenData.dir}__${file.name}__${this.random_string(10)}`
      param.append('key', key)
      param.append('policy', this.tokenData.policy)
      param.append('OSSAccessKeyId', this.tokenData.accessKey)
      param.append('success_action_status', 200)
      param.append('signature', this.tokenData.signature)
      param.append('file', file, file.name)
      // let url = 'http://test-yanglao-xuwenjie.oss-cn-beijing-internal.aliyuncs.com'
      const url = '/p1'
      axios({
        method: 'post',
        url: url,
        data: param,
        onUploadProgress: event => {
          content.file.percent = (event.loaded / event.total) * 100
          content.onProgress(content.file)
        }
      })
        .then(res => {
          content.onSuccess(key)
        })
        .catch(() => {
          content.onError()
        })
    },
    // 生成随机数
    random_string (len) {
      len = len || 32
      var chars = 'ABCDEFGHJKMNPQRSTWXYZabcdefhijkmnprstwxyz2345678'
      var maxPos = chars.length
      var pwd = ''
      for (let i = 0; i < len; i++) {
        pwd += chars.charAt(Math.floor(Math.random() * maxPos))
      }
      return pwd
    },
    // 预览
    handlePreview (file) {
      this.previewImage = file.url || file.thumbUrl
      this.previewVisible = true
      this.$emit('onPreview', file)
    },
    handleChange (file, fileList) {},
    // 删除
    handleRemove (file, list) {
      const name = file.name
      let finditem
      let index = -1
      for (const item of this.keyList) {
        if (item.indexOf(name) !== -1) {
          finditem = item
        }
      }
      if (finditem) {
        index = this.keyList.indexOf(finditem)
      }
      if (index !== -1) {
        this.keyList.splice(index, 1)
        this.$emit('onChange', this.keyList.join(','))
      }
    },
    // 进度条
    handleProgress (file) {},
    // 上传成功
    handleSuccess (key) {
      this.keyList.push(key)
      this.$emit('onChange', this.keyList.join(','))
    },
    // 超出数量限制
    handleExceed () {
      this.$message.error(`只可上传${this.limit}份文件`)
      this.$emit('onExceed', this.limit)
    }
  }
}
</script>

<style lang="less" scoped>
.upload-wrapper /deep/ .el-upload__input {
  display: none;
}
</style>
