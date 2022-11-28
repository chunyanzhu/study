#vue2三种方式修改props

###1.emit使用回调函数
```
//child
<template>
    <Input @on-change="handleEvent"></Input>
</template>
<script>
export default {
    name: "SearchParams",
    props:{
        // 信息文本
        message: {
            type: String,
            require: true,
            default: ()=> ''
        }
    },
    methods: {
        handleEvent(value){
            // 直接赋值报错
            // this.message = value.data
            // 调用回调函数并传值给父组件
            this.$emit('callBack', value.data);
        }
    },
}
</script>


//parent
<template>
    <div class="modules">
        <section class="modules-item">
            <h2>vue子组件修改父组件绑定数据</h2>
            <search-params :message="message" @callBack="callBack"/>
            <p>信息内容：{{ message }}</p>
        </section>
    </div>
</template>

<script>
import SearchParams from '@/components/search-params';
export default {
    components:{ SearchParams },
    data() {
       return {
          message: '我是信息文本',
       };
    },
    watch: {
       message: {
          handler (n, o) {
              console.log('修改前：' + n, '修改后：' + o);
          },
          deep: true,
          immediate: true
       }
    },
    methods:{
       callBack (msg) {
           console.log('回调函数值：' + msg);
     }
   }
}
</script>

```
###2.使用v-model双向绑定
v-model不只是表单元素input/form/select特有的，自己封装的组件也可以使用
v-model的原理，在编译的时候修改input
<input :value="" @change="">
```
// child
<template>
    <Input @on-change="handleEvent"></Input>
</template>
<script>
export default {
    name: "SearchParams",
    model: { // 绑定参数父组件传递变量
        prop: 'message',
        event: 'onChange'
    },
    props:{
        // 信息文本
        message: {
            type: String,
            require: true,
            default: ()=> ''
        }
    },
    methods: {
        handleEvent(value){
            // 直接赋值报错
            // this.message = value.data
            // 触发 model 绑定事件 onChange，实时传递变化值到父组件
            this.$emit('onChange', value.data)
        }
    },
}
</script>

// parent
<template>
    <div class="modules">
        <section class="modules-item">
            <h2>vue子组件修改父组件绑定数据</h2>
            <!-- 使用v-model绑定参数 -->
            <search-params v-model="message"/>
            <p>信息内容：{{ message }}</p>
        </section>
    </div>
</template>
```
###3.变量使用sync方式
```
// parent
<search-params :message.sync="message" >

//child
handleEvent(value){
    // value.data 获取输入框的值
    this.$emit('update:message', value.data)
}
```