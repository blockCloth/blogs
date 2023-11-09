## 初识VUE

### 初识Vue

1. 若想要让Vue工作，就必须创建一个Vue实例，且要传入一个配置对象
2. 可以通过ID来获取容器，容器里的代码被称为【Vue模板】
3. Vue实例和容器是一 一对应的
4. {{xxx}}中的xxx要写JS表达式，且xxx可以自动读取到data中的所有属性

```vue
<div id="demo">
    <h1>Hello，{{name}}，{{address}}</h1>
</div>

<script type="text/javascript" >
    Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。

    //创建Vue实例
    new Vue({
        el:'#demo', //el用于指定当前Vue实例为哪个容器服务，值通常为css选择器字符串。
        data:{ //data中用于存储数据，数据供el所指定的容器去使用，值我们暂时先写成一个对象。
            name:'atguigu',
            address:'北京'
        }
    })

</script>
```



### Vue模板语法

1. 插值语法
   - 功能：用于解析标签体内容
   - 写法：{{xxx}}，xxx是JS表达式，且可以直接读取到data中的所有属性
2. 执行写法
   - 功能：用于解析标签（包括：标签属性、标签体内容、绑定事件.....）
   - v-bind，绑定配置对象中的属性数据绑定

```vue
<div id="root">
    <h1>插值语法</h1>
    <h3>你好，{{name}}</h3>
    <hr/>
    <h1>指令语法</h1>
    <a v-bind:href="school.url.toUpperCase()" x="hello">点我去{{school.name}}学习1</a>
    <a :href="school.url" /* : === v-bind*/>点我去{{school.name}}学习2</a>
</div>

<script type="text/javascript">
    Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。

    new Vue({
        el:'#root',
        data:{
            name:'jack',
            school:{
                name:'尚硅谷',
                url:'http://www.atguigu.com',
            }
        }
    })
</script>
```

### 数据绑定

Vue中有两种数据绑定的方式：

1. 单向绑定（v-bind）：数据只能从data流向页面
2. 双向绑定（v-model）：数据不仅能从data流向页面，还可以从页面流向data
   1. 双向绑定一般都应用在表单类元素上（如：input、select等）
   2. v-model:value 可以简写为v-model，因为v-model 默认收集的就是value值

```vue
<div id="root">
    <!-- 普通写法 -->
    <!-- 单向数据绑定：<input type="text" v-bind:value="name"><br/>
    双向数据绑定：<input type="text" v-model:value="name"><br/> -->

    <!-- 简写 -->
    单向数据绑定：<input type="text" :value="name"><br/>
    双向数据绑定：<input type="text" v-model="name"><br/>

    <!-- 如下代码是错误的，因为v-model只能应用在表单类元素（输入类元素）上 -->
    <!-- <h2 v-model:x="name">你好啊</h2> -->
</div>

<script type="text/javascript">
    Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。

    new Vue({
        el:'#root',
        data:{
            name:'尚硅谷'
        }
    })
</script>
```

### Data与el的2种写法

1. el有2种写法
   1. new Vue时候配置el属性。
   2. 先创建Vue实例，随后再通过vm.$mount('#root')指定el的值。
2. data有2种写法
   1. 对象式
   2. 函数式
   3. 如何选择：目前哪种写法都可以，以后学习到组件时，data必须使用函数式，否则会报错。
3. 一个重要的原则：
   1. 由Vue管理的函数，一定不要写箭头函数，一旦写了箭头函数，this就不再是Vue实例了。

```vue
<script type="text/javascript">
    Vue.config.productionTip = false //阻止 vue 在启动时生成生产提示。

    //el的两种写法
    /* const v = new Vue({
			//el:'#root', //第一种写法
			data:{
				name:'尚硅谷'
			}
		})
		console.log(v)
		v.$mount('#root') //第二种写法 */

    //data的两种写法
    new Vue({
        el:'#root',
        //data的第一种写法：对象式
        /* data:{
				name:'尚硅谷'
			} */

        //data的第二种写法：函数式
        data(){
            console.log('@@@',this) //此处的this是Vue实例对象
            return{
                name:'尚硅谷'
            }
        }
    })
</script>
```

