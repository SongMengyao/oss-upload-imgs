# oss-upload-imgs
## 上传组件使用说明


## 参考文档
[element-upload组件](https://element.eleme.cn/#/zh-CN/component/upload)

[阿里云oss上传说明](https://help.aliyun.com/document_detail/31927.html?spm=a2c4g.11186623.6.1328.6dd558abOX5z1H)


## 参数说明

##### type : 可传文件后缀名

> 类型 : String

> 是否必须: 否

> 默认值 : '.jpg, .jpeg, .png, .gif, .bmp, .JPG, .JPEG, .PBG, .GIF, .BMP'

> 可选值 : .jpg, .jpeg, .png, .gif, .bmp, .JPG, .JPEG, .PBG, .GIF, .BMP, .doc, .pdf, .txt, .zip, .rm, .exe等等
***

##### listType : 文件列表的类型

> 类型 : String

> 是否必须 : 否

> 默认值 : 'picture-card'

> 可选值 : text / picture / picture-card
***

##### limit : 最大允许上传文件个数

> 类型 : Number

> 是否必须 : 否

> 默认值 : 1

> 可选值 : 任意数字
***

##### defaultList : 默认显示的已上传文件

> 类型 : String

> 是否必须 : 否

> 默认值 : ''

> 可选值 : 用 `,` 合并的文件url的字符串，文件url必须符合oss文件url规则

> 例如 : 
```
http://test-yanglao-xuwenjie.oss-cn-beijing-internal.aliyuncs.com/test/__food2.jpg__NHDFWSDR
```
***


## 可监听事件

##### onChange : 文件列表改变时的钩子，添加文件成功、删除文件成功会被调用

> 参数 : keyList

> 参数说明 : 用 `,`合并的文件key字符串

> 参数样例 : 
```
test/__food2.jpg__NHDFWSDR,test/__food1.jpg__JJSAWRRE
```
***

##### onPreview : 预览文件时的钩子

> 参数 : file对象

> 参数说明 : 文件对象

> 参数样例 : 
```
{
  uid: 177683322,
  url: 'http://ww.baidu.com/food2.jpg',
  size: 76345,
  name: 'food2.jpg'
}
```
***

##### onExceed : 文件超出个数限制时的钩子

> 参数 : limit

> 参数说明 : 限制的文件个数

> 参数样例 : 1
***


## 用法样例
> 同步加载 defaultList 或者不需 defaultList
```
<template>
  <upload
    @onChange="handleUploadChange">
  </upload>
</template>

<script>
import upload from '@/components/Upload/Upload'

export default {
  data() {
    return {
      Dish: {}
    }
  },
  methods: {
    handleUploadChange (keyList) {
      this.Dish.itemImg = keyList
    }
  }
}
</script>
```

> 异步请求 defaultList

```
<template>
  <upload
    v-if="showUpload"
    :defaultList="Dish.itemImg"
    @onChange="handleUploadChange">
  </upload>
</template>

<script>
import upload from '@/components/Upload/Upload'

export default {
  data() {
    return {
      showUpload: false,
      Dish: {}
    }
  },
  created() {
    this.$dispatch('getDishData')
    .then((res) => {
      this.showUpload = true
      this.Dish = res
    })
  },
  methods: {
    handleUploadChange (keyList) {
      this.Dish.itemImg = keyList
    }
  }
}
</script>
```
