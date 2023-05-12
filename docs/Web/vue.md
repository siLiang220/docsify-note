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