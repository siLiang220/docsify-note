## 组件通信方式

### 1. props 实现父子组件通信，props 的数据是只读的

1. 需要使用 `defineProps` 方法接收父组件传入的数据
2. `defineProps` 是vue3提供的方法不需要引入

-  父组件中向子组件传递数据
```js
<template>
  <div class="box">
    <h1>props:我是父组件曹操</h1>
    <hr />
    <Child info="我是曹操" :money="money"></Child>
  </div>
</template>

<script setup lang="ts">
//props:可以实现父子组件通信,props数据还是只读的！！！
import Child from "./Child.vue";
import { ref } from "vue";
let money = ref(10000);
</script>
```

- 子组件接收父组件的数据
```js
<template>
  <div class="son">
       <h1>我是子组件:曹植</h1>
       <p>{{props.info}}</p>
       <p>{{props.money}}</p>
      <!--props可以省略前面的名字--->
       <p>{{info}}</p>
       <p>{{money}}</p>
       <button @click="updateProps">修改props数据</button>
  </div>
</template>

<script setup lang="ts">
//需要使用到defineProps方法去接受父组件传递过来的数据
//defineProps是Vue3提供方法,不需要引入直接使用
let props = defineProps(['info','money'])//数组|对象写法都可以
//按钮点击的回调
const updateProps = ()=>{
  // props.money+=10;  props:只读的
  console.log(props.info)
}
```


### 2. defineEmits  子组件向父组件传递

`defineEmits`在使用时无需引入可以直接使用

1. 在子组件中调用 `defineEmits` 定义要发送给父组件的方法
```js
let $emit = defineEmits(['xxx','click']);
```
2. 使用`defineEmits` 会返回一个方法，使用一个变量$emit去接收
3. 在组件要发射的方法中调用$emit方法并将发送给父组件的方法和参数传入
```js
const handler = () => {
  //第一个参数:事件类型 第二个|三个|N参数即为注入数据
    $emit('xxx','东风导弹','航母');
};
```


### 3. eventBus 兄弟组件传值
1. 引入mitt 插件
```shell
npm install mitt- S
```
2. mitt 一个方法，方法执行的时候会返回一个bus对象
```js
import mitt from 'mitt';
const $bus = mitt();
export default $bus;
```
3. 在组件中相互使用 emit 派发事件， on监听事件，off 方法移除事件，clear 清除事件 

A组件监听（on）
```js
<template>
  <div class="child1">
    <h3>我是子组件1:曹植</h3>
  </div>
</template>

<script setup lang="ts">
import $bus from "../../bus";
//组合式API函数
import { onMounted } from "vue";
//组件挂载完毕的时候,当前组件绑定一个事件,接受将来兄弟组件传递的数据
onMounted(() => {
  //第一个参数:即为事件类型  第二个参数:即为事件回调
  $bus.on("car", (car) => {
    console.log(car);
  });
});
</script>

```

B组件派发（emit）
```js
<template>
  <div class="child2">
     <h2>我是子组件2:曹丕</h2>
     <button @click="handler">点击我给兄弟送一台法拉利</button>
  </div>
</template>

<script setup lang="ts">
//引入$bus对象
import $bus from '../../bus';
//点击按钮回调
const handler = ()=>{
  $bus.emit('car',{car:"法拉利"});
}
</script>
```
