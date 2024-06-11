**`element` 中`select` 组件中如果选中的值没有发生变化时，change方法是不会触发的**
![](https://zhaosi-1253759587.cos.ap-nanjing.myqcloud.com/files/obsidian/picture/20230512164843.png)
实现的方式 `<el-options>`   标签中绑定原生的`natite`方法
```html
<el-select
v-model="status"
placeholder="请选择"
style="margin-right: 10px"
>
<el-option
  v-for="item in workOrdersStauts"
  :key="item.value"
  :label="item.label"
  :value="item.status"
  @click.native="workOrderList('all')"
>
</el-option>
</el-select>
```


## 前端图片压缩组件

### shrinkpng 图片压缩

- [git地址](https://github.com/GHkmmm/shrinkpng)

示例代码
```js
<template>
  <div class="upload-container">
    <div class="upload-wrapper">
      <a-upload
        v-if="urlList.length < maxCount"
        :maxCount="maxCount"
        :customRequest="customRequest"
        :beforeUpload="beforeUpload"
        list-type="picture-card"
        :disabled="disabled"
        :fileList="[]"
        :multiple="multiple"
      >
        <div v-if="loading">
          <loading-outlined />
          <div class="ant-upload-text">上传中</div>
        </div>
        <div v-else>
          <plus-outlined />
          <div class="ant-upload-text">上传</div>
        </div>
      </a-upload>
    </div>
    <div class="images-container">
      <template v-if="urlList.length">
        <div class="img" v-for="(url, index) in urlList" :key="index">
          <a-image :width="90" :height="90" :src="`${domain}/${url}`">
            <template #previewMask>
              <a-space>
                <eye-outlined />
                <delete-outlined @click.stop="handleRemove(index)" v-if="!disabled" />
              </a-space>
            </template>
          </a-image>
        </div>
      </template>
    </div>
  </div>
</template>

<script setup lang="ts">
  import { PlusOutlined, LoadingOutlined, EyeOutlined, DeleteOutlined } from '@ant-design/icons-vue'
  import { upload, del } from '/@/api/system/upload/index'
  import { ref, watch } from 'vue'
  import { useMessage } from '/@/hooks/web/useMessage'
  import { FilePathEnum } from '/@/enums/FilePathEnum'
  import { shrinkImage } from "shrinkpng";

  const { createMessage } = useMessage()

  // 定义 props
  const props = defineProps({
    modelValue: {
      type: Array<string>,
      default: [],
    },
    validType: {
      type: Array,
      default: () => ['JPG', 'JPEG', 'PNG', 'GIF'],
    },
    maxCount: {
      type: Number,
      default: 3,
    },
    disabled:{
      type: Boolean,
      defaule: true
    },
    multiple:{
      type:Boolean,
      default: false
    }
  })

  // 定义 emits
  const emit = defineEmits(['update:modelValue'])
  const domain = ref<string>('https://iot-bintaike-oss.oss-cn-shanghai.aliyuncs.com')
  // 定义引用
  const disabled = ref<boolean>(false)
  const multipe = ref<boolean>(false)
  const loading = ref(false)
  const urlList = ref<string[]>([])
  const customRequest = async ({ file }: { file: File }) => {

    //对文件进行压缩
    const _file = await shrinkImage(file, {
      quality: 50
    });

    loading.value = true
    const formData = new FormData()
    formData.append('file', _file)
    formData.append('biz', FilePathEnum.RICH)

    const res = await upload(formData)
    urlList.value.push(res.filePath)
    loading.value = false
    createMessage.success('上传成功')
  }

  function handleFileSuffix(name: string) {
    return name.substring(name.lastIndexOf('.') + 1).toUpperCase()
  }

  function beforeUpload(file: File) {
    const suffix = handleFileSuffix(file.name)
    if (props.validType.length) {
      const isValidType = props.validType.includes(suffix)
      if (!isValidType) {
        createMessage.error(`【${file.name}】文件格式不合法!`)
        return false
      }
    }
    const isLt2M = file.size / 1024 / 1024 < 25
    if (!isLt2M) {
      createMessage.error('文件超过 10MB!请压缩图片')
      return false
    }
    return true
  }

  // function isFinish() {
  //   return loading.value
  // }

  function handleRemove(index: number) {
    // createMessage.info()
    // createMessage.confirm({
    //   title: '确认删除?',
    //   onOk() {
    //     urlList.value.splice(index, 1)
    //   },
    // })

    del(urlList.value[index])
    urlList.value.splice(index, 1)
    createMessage.success('删除成功')
  }

  // 监听 props 的变化
  watch(
    () => props.modelValue,
    (value) => {
      urlList.value = value
    },
    { immediate: true },
  )
  watch(
    () => props.disabled,
    (value) => {
      disabled.value = value
    },
    { immediate: true },
  )
  watch(
    ()=>props.multiple,
    (value)=>{
      multipe.value = value
    },
    { immediate: true },
  )
  // 监听 urlList 变化并更新 modelValue
  watch(
    urlList,
    (value) => {
      emit('update:modelValue',value)
    },
    { deep: true },
  )
</script>

<style scoped lang="less">
  .upload-container {
    display: flex;
    flex-wrap: wrap;
    justify-content: flex-start;
  }

  .upload-wrapper {
    order: 1;
  }

  .images-container {
    display: flex;
    flex-wrap: wrap;
    justify-content: flex-start;
  }

  .img {
    width: 102px;
    height: 102px;
    border: 1px dashed #d9d9d9;
    padding: 4px;
    display: flex;
    align-items: center;
    justify-content: center;
    margin-bottom: 10px;
    margin-right: 10px;

    ::v-deep(.ant-image) {
      display: flex;
      align-items: center;
      overflow: hidden;
    }
  }
</style>

```

## Vue打印机打印组件

### vue-print-nb
- [仓库地址](https://www.npmjs.com/package/vue-print-nb#vue3-version)