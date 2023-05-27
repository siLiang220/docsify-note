## 一、组件通信方式

### 1. props 实现父子组件通信，props 的数据是只读的

1. 需要使用 `defineProps` 方法接收父组件传入的数据
2. `defineProps` 是vue3提供的方法不需要引入

父组件中向子组件传递数据
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

子组件接收父组件的数据

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

### 4. v-mode实现父子组件通信

父组件通过 “v-model:绑定的属性名” 传递数据属性，支持绑定多个属性；可以实现父子组件数据同步的业务
```js
// 父组件
// v-model 没有指定参数名时，子组件默认参数名是modelValue
<Child v-model="search" />
```

父组件
```js
<template>
  <div>
    <h1>v-model:钱数{{ money }}{{pageNo}}{{pageSize}}</h1>
    <input type="text" v-model="info" />
    <hr />
    <!-- props:父亲给儿子数据 -->
    <!-- <Child :modelValue="money" @update:modelValue="handler"></Child> -->
    <!-- 
       v-model组件身上使用
       第一:相当有给子组件传递props[modelValue] = 10000
       第二:相当于给子组件绑定自定义默认事件update:modelValue
     -->
    <Child v-model="money"></Child>
    <hr />
    <Child1 v-model:pageNo="pageNo" v-model:pageSize="pageSize"></Child1>
  </div>
</template>

<script setup lang="ts">
//v-model指令:收集表单数据,数据双向绑定
//v-model也可以实现组件之间的通信,实现父子组件数据同步的业务
//父亲给子组件数据 props
//子组件给父组件数据 自定义事件
//引入子组件
import Child from "./Child.vue";
import Child1 from "./Child1.vue";
import { ref } from "vue";
let info = ref("");
//父组件的数据钱数
let money = ref(10000);
//自定义事件的回调
const handler = (num) => {
  //将来接受子组件传递过来的数据
  money.value = num;
};

//父亲的数据
let pageNo = ref(1);
let pageSize = ref(3);
</script>
```

子组件child
```js
<template>
  <div class="child">
    <h3>钱数:{{ modelValue }}</h3>
    <button @click="handler">父子组件数据同步</button>
  </div>
</template>

<script setup lang="ts">
//接受props
let props = defineProps(["modelValue"]);
let $emit = defineEmits(['update:modelValue']);
//子组件内部按钮的点击回调
const handler = ()=>{
   //触发自定义事件
   $emit('update:modelValue',props.modelValue+1000);
}
```

子组件child2
```js
<template>
  <div class="child2">
    <h1>同时绑定多个v-model</h1>
    <button @click="handler">pageNo{{ pageNo }}</button>
    <button @click="$emit('update:pageSize', pageSize + 4)">
      pageSize{{ pageSize }}
    </button>
  </div>
</template>

<script setup lang="ts">
let props = defineProps(["pageNo", "pageSize"]);
let $emit = defineEmits(["update:pageNo", "update:pageSize"]);
//第一个按钮的事件回调
const handler = () => {
  $emit("update:pageNo", props.pageNo + 3);
};
</script>
```

### 5. useAttrs 可以获取组件身上的属性和事件

`props`与`useAttrs`都可以获取父组件传过来的属性和值但是props接收了`useAttrs`就无法接收了

父组件中引入自定义的子组件
```js
<template>
  <div>
    <h1>useAttrs</h1>
    <el-button type="primary" size="small" :icon="Edit"></el-button>
    <!-- 自定义组件 -->
    <HintButton type="primary" size="small" :icon="Edit" title="编辑按钮" @click="handler" @xxx="handler"></HintButton>
  </div>
</template>

<script setup lang="ts">
//vue3框架提供一个方法useAttrs方法,它可以获取组件身上的属性与事件！！！
//图标组件
import {
  Check,
  Delete,
  Edit,
  Message,
  Search,
  Star,
} from "@element-plus/icons-vue";
import HintButton from "./HintButton.vue";
//按钮点击的回调
const handler = ()=>{
  alert(12306);
}
</script>
```

子组件通过`useAttrs` 方法获取组件标签身上的属性和方法
```js
<template>
  <div :title="title">
     <el-button :="$attrs"></el-button>   
  </div>
</template>

<script setup lang="ts">
//引入useAttrs方法:获取组件标签身上属性与事件
import {useAttrs} from 'vue';
//此方法执行会返回一个对象
let $attrs = useAttrs();

//万一用props接受title
let props =defineProps(['title']);
//props与useAttrs方法都可以获取父组件传递过来的属性与属性值
//但是props接受了useAttrs方法就获取不到了
console.log($attrs);
</script>
```

### 6. ref可以获取到真实的DOM节点，可以获取到子组件的VC

组件内部的数据和方法 是封闭的，外部组件是不可以访问。可以通过`defineExpose`方法对外暴露

父组件使用ref修改子组件的数据
```js
<template>
  <div class="box">
    <h1>我是父亲曹操:{{money}}</h1>
    <button @click="handler">找我的儿子曹植借10元</button>
    <hr>
    <Son ref="son"></Son>
    <hr>
    <Dau></Dau>
  </div>
</template>

<script setup lang="ts">
//ref:可以获取真实的DOM节点,可以获取到子组件实例VC
//$parent:可以在子组件内部获取到父组件的实例
//引入子组件
import Son from './Son.vue'
import Dau from './Daughter.vue'
import {ref} from 'vue';
//父组件钱数
let money = ref(100000000);
//获取子组件的实例
let son = ref();
//父组件内部按钮点击回调
const handler = ()=>{
   money.value+=10;
   //儿子钱数减去10
   son.value.money-=10;
   son.value.fly();
}
//对外暴露
defineExpose({
   money
})
</script>
```

子组件需要对外暴露内部数据和方法
```js
<template>
  <div class="son">
    <h3>我是子组件:曹植{{money}}</h3>
  </div>
</template>

<script setup lang="ts">
import {ref} from 'vue';
//儿子钱数
let money = ref(666);
const fly = ()=>{
  console.log('我可以飞');
}
//组件内部数据对外关闭的，别人不能访问
//如果想让外部访问需要通过defineExpose方法对外暴露
defineExpose({
  money,
  fly
})
</script>
```
### 7. $parent可以获取到父组件的数据

子组件修改父组件的数据在方法中使用`$parent`，且父组件需要对外暴露属性值
```js
<template>
  <div class="dau">
     <h1>我是闺女曹杰{{money}}</h1>
     <button @click="handler($parent)">点击我爸爸给我10000元</button>
  </div>
</template>

<script setup lang="ts">
import {ref} from 'vue';
//闺女钱数
let money = ref(999999);
//闺女按钮点击回调
const handler = ($parent)=>{
   money.value+=10000;
   $parent.money-=10000;
}
</script>
```
