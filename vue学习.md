# 创建VUE工程

基于vite创建

- 轻量快速的热重载（HMR），能快速启动服务
- 按需编译，不再等待整个应用编译完成
- 对TypeScript，CSS，JSX提供更好支持

# 组合式API

## setup

是一个函数

```vue
<template>
	<div class ="person">//.person样式没写出来
    	<p>{{name}}</p>
        <p>{{age}}</p>
    </div>
</template>
<script lang="ts">
    export default{//将内容公开
        name:'Person',//组件名
        setup(){
            //数据
            let name = '张三'//非响应式
            let age = 18//非响应式
            let tel = 'xxx'
            //方法
            function changeName(){
                name = '李四'
            }
            function changeAge(){
                age +=1
            }
            function showTel(){
                alert(tel)
            }
            return {name,age,changeName,changeAge,showTel}//交出的数据和方法
        }
    }
</script>
```



setup函数中的this是undefined

setup不能读取vue2写法的数据

**响应式：**会实时更新

### 自动return

```vue
<script lang='ts'>//第一个script用于定义组件名
    export default{
        name:'Person'
    }
</script>  
<script lang='ts' setup>
    let a = 666
</script>
```



# 响应式数据

## ref函数

主要用于基本类型响应式数据

```vue
<script lang='ts' setup>
    import {ref} from 'vue'
	//数据
    let name = ref('张三')//ref包裹
    let age = ref(18)
    let tel = 'xxx'
    //方法
    function changeName(){
        name.value = '李四'//使用ref要加.value
    }
    function changeAge(){
        age.value +=1
    }
    function showTel(){
        alert(tel)
    }
</script>
```

也可以用于引用数据类型，但实际上是使用了reactive函数

```vue
<template>
	<div class ="person">//.person样式没写出来
    	<p>一辆{{car.brand}},价值{{car.price}}万</p>
        <br>
        <p>游戏列表：</p>
        <ul>
            <li v-for="g in games" :key="g.id">//:为绑定js语句,v-for是遍历语句
                {{g.name}}
    		</li>
    	</ul>
    </div>
</template>
<script lang='ts' setup>
    import {ref} from 'vue'
	let car = ref({//reactive包裹
        brand:'奔驰',
        price:100
    })
    let games = ref([
        {
            id:'001',
            name:'王者荣耀'
        },
        {
        	id:'002',
        	name:'原神'
        }
    ])
    function changePrice(){
        car.value.price +=10;//需要加.value，紧接在变量命之后
    }
    function changefirstGame(){
        games.value[0].name = 'xxx'
    }
</script>
```



## reactive函数

只能用于引用数据类型的响应式数据，对于引用数据中的引用数据（多层次的引用数据）都生效

```vue
<template>
	<div class ="person">//.person样式没写出来
    	<p>一辆{{car.brand}},价值{{car.price}}万</p>
        <br>
        <p>游戏列表：</p>
        <ul>
            <li v-for="g in games" :key="g.id">//:为绑定js语句,v-for是遍历语句
                {{g.name}}
    		</li>
    	</ul>
    </div>
</template>
<script lang='ts' setup>
    import {reactive} from 'vue'
	let car = reactive({//reactive包裹
        brand:'奔驰',
        price:100
    })
    let games = reactive([
        {
            id:'001',
            name:'王者荣耀'
        },
        {
        	id:'002',
        	name:'原神'
        }
    ])
    function changePrice(){
        car.price +=10;//不用加.value
    }
    function changefirstGame(){
        games[0].name = 'xxx'
    }
</script>
```

reactive后的对象被重新分配为一个新对象，会失去响应式。可以使用Object.assign去替换

```vue
<script>
    function changeCar(){
        Object.assign(car,{brand:'宝马',price:90})
    }
</script>
```

## toRefs函数

将一个reactive定义的对象分配给一个新对象时，使用toRefs函数将新对象变为ref响应式数据，且新对象的数据和源对象的数据同步更新

```vue
<script lang='ts' setup>
    import {reactive} from 'vue'
	let car = reactive({//reactive包裹
        brand:'奔驰',
        price:100
    })
	let {brand,price} = toRefs(car)
    let price = toRef(car,'price')
    function changePrice(){
        price.value +=10;//需要value,新对象的price和car.price是同步更新的
    }
</script>
```

## computed计算属性

`let xxx = computed(回调函数)`

返回的值是响应式数据

有缓存，数据不变，不会重复执行。

是只读的，不可将赋值后的xxx重新赋值

```vue
<template>
	<div class ="person">//.person样式没写出来
		姓:<input type="text" v-model="firstName">//v-model用于绑定变量
        <br>
        名:<input type="text" v-model="lastName">
        <br>
        全名:<span>{{fullName}}</span>
    </div>
</template>
<script lang='ts' setup>
    import {ref,computed} from 'vue'
    
	let firstName = ref('张')
    let lastName = ref('三')
    
    let fullName = computed(()=>{
        return firstName.value.slice(0,1).toUpperCase()+firstName.value.slice(1) + '-' + lastName.value//需要加.value
    })
    //如此写是可读可写的
    let fullName = computed(()=>{
        get(){
            return firstName.value.slice(0,1).toUpperCase()+firstName.value.slice(1) + '-' + lastName.value//需要加.value
        },
		set(val){//val参数接受重新赋值的值
            let [str1,str2] = val.split('-')
            firstName.value = str1
            lastName.value = str2
        }
    })
</script>
```

# watch函数

`let stopWatch = watch(数据,(新值[,旧值])=>{`

`}[,{配置对象}])`

返回值是一个函数，调用它会停止监视

监视数据的变化

- ref数据，不需要.value
- reactive数据
- 函数的返回值
- 包括上述内容的数组

## 监视基本数据类型

```vue
<template>
	<div class ="person">//.person样式没写出来
    </div>
</template>
<script lang='ts' setup>
    import {ref,watch} from 'vue'
    let age = ref(0)
    function addAge(){
        age.value +=1
    }
    let stopWatch = watch(age,(newVal,oldVal)=>{//这里不需要.value
        xxx
        if(xxx){
            stopWatch()
        }
    })
</script>
```

## 监视ref定义的引用数据类型

监视的是对象的地址值的变化。

若想监视对象内部属性的变化，需要开启深度监视:在匿名函数之后加`,{deep:true}`。但是属性改变时newVal和oldVal的值是一样的

```vue
<template>
	<div class ="person">//.person样式没写出来
        <h1>姓名:{{person.name}}</h1>
    </div>
</template>
<script lang='ts' setup>
    import {ref,watch} from 'vue'
	let person = ref({
        name:'张三',
        age:18
    })
    function changeName(){
        person.value.name += '~'
    }
    function changePerson(){
        person.value = {name:'李四',age:17}
    }
    let stopWatch = watch(person,(newVal,oldVal)=>{
        xxx
    },{deep:true})
</script>
```

## 监视reactive定义的引用数据类型

隐式创建深层监听，不可关闭

newVal和oldVal的值是一样的

```vue
<template>
	<div class ="person">//.person样式没写出来
        <h1>姓名:{{person.name}}</h1>
    </div>
</template>
<script lang='ts' setup>
    import {reactive,watch} from 'vue'
	let person = reactive({
        name:'张三',
        age:18
    })
    function changeName(){
        person.name += '~'
    }
    function changePerson(){
        Object.assign(person,{name:'李四',age:17})//reactive只能这样修改整个对象
    }
    let stopWatch = watch(person,(newVal,oldVal)=>{
        xxx
    })
</script>
```

## 监视reactive/ref定义的引用数据类型的某个属性

此时newVal和oldVal的值是不同的

若属性值不是引用数据类型，需要写成函数形式

若属性值是引用数据类型，可以直接写，但监视不了整体的变化。写成函数形式，监视不了引用数据类型中的内部属性的变化。

**要写成函数式再加开启深度监视就能全部监视**

```vue
<template>
	<div class ="person">//.person样式没写出来
        <h1>姓名:{{person.name}}</h1>
    </div>
</template>
<script lang='ts' setup>
    import {reactive,watch} from 'vue'
	let person = reactive({
        name:'张三',
        age:18
        car:{
        	c1:'奔驰',
        	c2:'宝马'
    	}
    })
    function changeName(){
        person.name += '~'
    }
    function changeC1(){
        person.car.c1='奥迪'
    }
    function changeCar(){
		person.car = {c1:'劳斯莱斯',c2:'迈巴赫'}
    }
    let stopWatch = watch(()=>person.name,(newVal,oldVal)=>{
        xxx
    })
    let stopWatch1 = watch(person.car,(newVal,oldVal)=>{
        xxx
    })
    let stopWatch2 = watch(()=>person.car,(newVal,oldVal)=>{
        xxx
    },{deep:true})
</script>
```

## 监视包括上述内容的数组

newVal,oldVal是数组

```vue
<template>
	<div class ="person">//.person样式没写出来
        <h1>姓名:{{person.name}}</h1>
    </div>
</template>
<script lang='ts' setup>
    import {reactive,watch} from 'vue'
	let person = reactive({
        name:'张三',
        age:18
        car:{
        	c1:'奔驰',
        	c2:'宝马'
    	}
    })
    function changeName(){
        person.name += '~'
    }
    function changeC1(){
        person.car.c1='奥迪'
    }
    function changeCar(){
		person.car = {c1:'劳斯莱斯',c2:'迈巴赫'}
    }
	let stopWatch = watch([()=>person.name,()=>person.car.c1],(newVal,oldVal)=>{
        xxx
    },{deep:true})
</script>
```

## watchEffect

watch只监视指定的，watchEffect会自动分析那些需要监视

`watchEffect(()=>{`

`})`

初始化时会立刻执行一次回调函数

```vue
<template>
	<div class ="person">//.person样式没写出来
        <h1>当前水温：{{temp}}</h1>
    </div>
</template>
<script lang='ts' setup>
    import {ref,watch,watchEffect} from 'vue'
    let temp = ref(10)
    function changeTemp(){
        temp+=5
    }
    watchEffect(()=>{
        if(temp.value >=60){
            console.log('xxx')
        }
    })
    
</script>
```

# 标签的ref属性

只在当前的vue组件作用域中生效

```vue
<template>
	<div class ="person">
        <h2 ref='t2'>xxx</h2>//把标签用ref标记
    </div>
</template>
<script lang='ts' setup>
    import {ref} from 'vue'
    //创建一个容器,用于存储ref标记的内容
    let t2 = ref()
	function showLog(){
        console.log(t2.value)//输出xxx
    }
</script>
<style scoped>//scoped使得样式只在当前vue组件生效
</style>
```



放在组件标签中会获得组件的实例

```vue
<template>
	<div class ="person">
        <h2>xxx</h2>//把标签用ref标记
    </div>
</template>
<script lang='ts' setup>
    import {ref} from 'vue'
    //创建一个容器,用于存储ref标记的内容
    let t2 = ref()
	function showLog(){
        console.log(t2.value)//输出xxx
    }
    defineExpose({t2})
</script>
```

加上defineExpose函数，用于组件给出内容

# propos

用于接收组件间传递的数据。

接收方先引入definePropos，再使用definePropos接收传递过来的数据，返回的是对象数组

可以只接收一部分,也可以加上泛型用于限制传入类型，也可以限定必要性（是否必须传入），还可以只当默认值

传递方：

```vue
<template>
	<Person a='x1' :list = "personList"/>
</template>
<script lang='ts' setup>
    import {reactive} from 'vue'
    import Person from '/Person.vue'
	let x = 9
    let personList = reactive([
        {...},
        {...}
    ])
</script>
```

接收方：

```vue
<template>
	<div class ="person">
        <h2>{{list}}</h2>//把标签用ref标记
    </div>
</template>
<script lang='ts' setup>
    import {reactive,withDefaults} from 'vue'
    let x = definePropos(['a'])//{a:'x1'}
    definePropos<list:xxx>()//xxx是一个自定义类型
    definePropos<list?:xxx>()//?用于让它不必要
    withDefaults(definePropos<list?:xxx>(),{
        list:()=>[{...}]
    })//withDefaults用于指定默认值
</script>
```

**define开头的函数不需要引入**

# 组件的生命周期

1. 创建
2. 挂载mount
3. 更新
4. 卸载

## 挂载

使用onBeforeMount指定挂载前调用的函数

使用onMounted指定挂载完毕调用的函数

```vue
<template>
	<div class ="person">
        <h2>{{sum}}</h2>//把标签用ref标记
    </div>
</template>
<script lang='ts' setup>
    import {ref,onBeforeMount,onMounted} from 'vue'
	let sum = ref(0)
    function add(){
        sum+=1
    }
    onBeforeMount(()=>{
        xxx
    })
    onMounted(()=>{
        xxx
    })
</script>
```

## 更新

使用onBeforeUpdate指定挂载前调用的函数

使用onUpdated指定挂载完毕调用的函数

用法同上

## 卸载

使用onBeforeUnmount指定挂载前调用的函数

使用onUnmounted指定挂载完毕调用的函数

用法同上

# 自定义hook

hook用于模块化封装

以use+功能命名的.js/.ts文件

不用hook：

```vue
<template>
	<div class ="person">
        <h2>{{sum}}</h2>//把标签用ref标记
    </div>
	<hr>
	<img v-for = "(dog,index) in dogList" :src="dog" :key="index">
</template>
<script lang='ts' setup>
    import {ref,reactive} from 'vue'
    import axios from 'axios'
	let sum = ref(0)
    let dogList = reactive(['xxx'])
    function add(){
        sum+=1
    }
    async function getDog(){
        try{
            let result = await axios.get('yyy')
        	dogList.push(result.data.zzz)
        }catch(error){
            alert(error)
        }
    }
</script>
<style>
    .person{
        
    }
    img{
        height:100px;
    }
</style>
```

用hook：

useDog.ts

```typescript
import {reactive} from 'vue'
import axios from 'axios'
export default function(){
    let dogList = reactive(['xxx'])
    async function getDog(){
        try{
            let result = await axios.get('yyy')
            dogList.push(result.data.zzz)
        }catch(error){
            alert(error)
        }
    }
    return {dogList,getDog}//向外交出数据和方法
}
```

useSum.ts

```typescript
import {ref} from 'vue'
export default function(){
    let sum = ref(0)
    function add(){
        sum+=1
    }
    return {sum,add}
}
```

vue组件

```vue
<template>
	<div class ="person">
        <h2>{{sum}}</h2>//把标签用ref标记
    </div>
	<hr>
	<img v-for = "(dog,index) in dogList" :src="dog" :key="index">
</template>
<script lang='ts' setup>
	import useSum from '@/hooks/useSum'
    import useDog from '@/hooks/useDog'
    const {sum,add} = useSum()
    const {dogList,getDog} = useDog()
</script>
```

# 路由

SPA：单页面应用

路由规则：/xxx对应xxx.vue

```typescript
import {createRouter,createWebHistory} from 'vue-router'//引入路由
import x1 from '@/components/x1.vue'
import x2 from '@/components/x2.vue'
import x3 from '@/components/x3.vue'
//创建路由器
const router = createRouter({
    history:createWebHistory(),//路由器的工作模式
    routes:[
        {path:'/x1',component:x1},
        {path:'/x2',component:x2},
        {path:'/x3',component:x3},
    ]
})

export default router//交出router
//-------------------main.ts中------------

import router from '/router'
const app = createApp(App)
app.ues(router)//使用路由器
app.mount('#app')
//-------------------App.vue中------------
<template>
	<div class ="app">
        <div class="navigate">
            <RouterLink to="/x1">z1</RouterLink>
			<RouterLink to="/x2">z2</RouterLink>
    </div>
</template>
<script lang='ts' setup>
	import {RouterView,RouterLink} from 'vue-router'
</script>
```

注意：

- 路由组件通常放在pages或views文件夹，一般组件存放在components文件夹
- 通过点击导航，视觉效果上小时的路由组件，默认是被卸载的，需要的时候再去挂载

一般组件：程序员亲手写标签`<xxx/>`

路由组件：靠路由的规则渲染出来

## 路由器工作模式

### history模式

`history:createWebHistory`

优点：URL更加美观，不带有#

缺点：需要服务端配合处理路径问题

### hash模式

`history:createWebHashHistory`

优点：兼容性更好，不需要服务端配合处理路径问题

缺点：URL带有#，且在SEO优化较差

## to的写法

字符串写法

```vue
<template>
	<div class ="app">
        <div class="navigate">
            <RouterLink to="/x1">z1</RouterLink>
			<RouterLink to="/x2">z2</RouterLink>
    </div>
</template>
<script lang='ts' setup>
	import {RouterView,RouterLink} from 'vue-router'
</script>
```

对象写法

```vue
<template>
	<div class ="app">
        <div class="navigate">
            <RouterLink :to="{path:'/x1'}">z1</RouterLink>
			<RouterLink :to="{path:'/x2'}">z2</RouterLink>
    </div>
</template>
<script lang='ts' setup>
	import {RouterView,RouterLink} from 'vue-router'
</script>
```

## 命名路由

```typescript
import {createRouter,createWebHistory} from 'vue-router'//引入路由
import x1 from '@/components/x1.vue'
import x2 from '@/components/x2.vue'
import x3 from '@/components/x3.vue'
//创建路由器
const router = createRouter({
    history:createWebHistory(),//路由器的工作模式
    routes:[
        {name:'x1',path:'/x1',component:x1},
        {name:'x2',path:'/x2',component:x2},
        {name:'x3',path:'/x3',component:x3}
    ]
})

export default router//交出router
```

App.vue中

```vue
<template>
	<div class ="app">
        <div class="navigate">
            <RouterLink :to="{name:'x1'}">x1</RouterLink>//名字跳转
			<RouterLink :to="{path:'/x2'}">x2</RouterLink>//路径跳转
    </div>
</template>
<script lang='ts' setup>
	import {RouterView,RouterLink} from 'vue-router'
</script>
```

## 嵌套路由

给路由对象加上children属性

```typescript
import {createRouter,createWebHistory} from 'vue-router'//引入路由
import x1 from '@/components/x1.vue'
import x2 from '@/components/x2.vue'
import x3 from '@/components/x3.vue'
//创建路由器
const router = createRouter({
    history:createWebHistory(),//路由器的工作模式
    routes:[
        {name:'x1',path:'/x1',component:x1},
        {name:'x2',path:'/x2',component:x2,children:[{
            name:'x4',
            path:'x4',//子级不用写/
            component:x4
        }]},
        {name:'x3',path:'/x3',component:x3}
    ]
})

export default router//交出router
```

使用RouterLink时to的地址要`"/x2/x4"`这样写

```vue
<template>
	<div class ="app">
        <div class="navigate">
            <RouterLink to="/x1">z1</RouterLink>
			<RouterLink to="/x2/x4">z4</RouterLink>
    </div>
</template>
<script lang='ts' setup>
	import {RouterView,RouterLink} from 'vue-router'
</script>
```



## query参数

在RouterLink时to的地址后加上？表示开始传参，参数之间用&链接

```vue
<template>
	<div class ="app">
        <div class="navigate">
            <ul>
                <li v-for="news in newList" :key="news.id">
                    //第一种写法
                    <RouterLink :to="`/x2?id=${news.id}&b=${news.title}&c=${news.content}`">{{news.title}}</RouterLink>
                    //第二种写法
                    <RouterLink :to="{
                                     path:'/x2',//或者name:'x2',
                                     query:{id:news.id,title:news.title,content:news.content}
                                     }">
                        {{news.title}}
                    </RouterLink>
    		</li>
    	</ul>
    </div>
</template>
<script lang='ts' setup>
	import {RouterView,RouterLink} from 'vue-router'
</script>
```

x2.vue中。

使用参数需要加.query

```vue
<template>
	<ul class ="xxx">
        <li>编号：{{route.query.id}}</li>
        <li>标题：{{route.query.title}}</li>
        <li>内容：{{route.query.content}}</li>
    </ul>
</template>
<script lang='ts' setup>
	import {useRoute} from 'vue-router'
    const route = useRoute()
</script>
```

## params参数

不可传递对象和数组

```vue
<template>
	<div class ="app">
        <div class="navigate">
            <ul>
                <li v-for="news in newList" :key="news.id">
                    //第一种写法
                    <RouterLink :to="`/x2/${news.id}/${news.title}/${news.content}`">{{news.title}}</RouterLink>
                    //第二种写法
                     <RouterLink :to="{
                                      name:'x2',//只能用name
                                      params:{
                                      	id:news.id,
                                      	title:news.title,
                                      content:news.content
                                      }
                                      }">{{news.title}}</RouterLink>
    		</li>
    	</ul>
    </div>
</template>
<script lang='ts' setup>
	import {RouterView,RouterLink} from 'vue-router'
</script>
```

需要占位

可以在末尾加？配置参数的必要性

```typescript
import {createRouter,createWebHistory} from 'vue-router'//引入路由
import x1 from '@/components/x1.vue'
import x2 from '@/components/x2.vue'
import x3 from '@/components/x3.vue'
//创建路由器
const router = createRouter({
    history:createWebHistory(),//路由器的工作模式
    routes:[
        {name:'x1',path:'/x1',component:x1},
        {name:'x2',path:'/x2/:id/:title/:content?',component:x2,children:[{
            name:'yyy',
            path:'x4',//子级不用写/
            component:x4
        }]},
        {name:'x3',path:'/x3',component:x3}
    ]
})

export default router//交出router
```

x2.vue中

使用参数需要加.params

```vue
<template>
	<ul class ="xxx">
        <li>编号：{{route.params.id}}</li>
        <li>标题：{{route.params.title}}</li>
        <li>内容：{{route.params.content}}</li>
    </ul>
</template>
<script lang='ts' setup>
    import {useRoute} from 'vue-router'
    const route = useRoute()
</script>
```

## 路由规则的props配置

第一种写法：

在路由对象中加入props:true。将路由收到的所有**params参数**作为props传给路由组件

第二种写法：

在路由对象中加入。函数写法

```typescript
props(route){
    return route.query
}
```

自己决定将什么作为props传给路由组件

第三种写法：

在路由对象中加入。对象写法

```typescript
props:{
    id:200,
    title:200,
    content:200
}
```

自己决定将什么作为props传给路由组件，但只能传固定的值



```typescript
import {createRouter,createWebHistory} from 'vue-router'//引入路由
import x1 from '@/components/x1.vue'
import x2 from '@/components/x2.vue'
import x3 from '@/components/x3.vue'
//创建路由器
const router = createRouter({
    history:createWebHistory(),//路由器的工作模式
    routes:[
        {name:'x1',path:'/x1',component:x1},
        {name:'x2',path:'/x2/:id/:title/:content?',component:x2,props:true,children:[{
            name:'yyy',
            path:'x4',//子级不用写/
            component:x4
        }]},
        {name:'x3',path:'/x3',component:x3}
    ]
})

export default router//交出router
```

x2.vue中

```vue
<template>
	<ul class ="xxx">
        <li>编号：{{id}}</li>
        <li>标题：{{title}}</li>
        <li>内容：{{content}}</li>
    </ul>
</template>
<script lang='ts' setup>
    defineProps(['id','title','content'])
</script>
```

## replace属性

控制路由跳转时操作浏览器历史记录，替换模式，无法后退和前进

在RouterLink和to之间加上replace

```vue
<template>
	<div class ="app">
        <div class="navigate">
            <RouterLink replace to="/x1">z1</RouterLink>
			<RouterLink replace  to="/x2/x4">z4</RouterLink>
    </div>
</template>
<script lang='ts' setup>
	import {RouterView,RouterLink} from 'vue-router'
</script>
```

## 编程式导航

脱离`<RouterLink>`实现路由跳转

```vue
<script lang='ts' setup>
    import {onMounter} from 'vue'
	import {useRouter} from 'vue-router'
    const router = useRouter()
   	onMounted(()=>{
        setTimeout(()=>{
            router.push('/news')
        },3000)
    })
</script>
```

push里的参数和`<RouterLink>`中to的参数是相同格式

## 重定向

在路由规则中加入

```typescript
import {createRouter,createWebHistory} from 'vue-router'//引入路由
import x1 from '@/components/x1.vue'
import x2 from '@/components/x2.vue'
import x3 from '@/components/x3.vue'
//创建路由器
const router = createRouter({
    history:createWebHistory(),//路由器的工作模式
    routes:[
        {name:'x1',path:'/x1',component:x1},
        {name:'x2',path:'/x2/:id/:title/:content?',component:x2,props:true,children:[{
            name:'yyy',
            path:'x4',//子级不用写/
            component:x4
        }]},
        {name:'x3',path:'/x3',component:x3},
     	{
            path:'/',
            redirect:'/x1'
        }
    ]
})

export default router//交出router
```

# pinia

集中式数据（状态）管理

用于多个组件共享数据



先使用`npm i pinia`下载pinia

在src文件夹中的store文件夹创建pinia文件，比如count.ts

在app.vue中

```vue
<template>
	<Count/>
</template>
<script lang='ts' setup>
	import Count from './component/Count.vue'
</script>
```

在Count.vue中

```vue
<template>
	<h2>当前求和为：{{sum}}</h2>
	<select v-model.number="n">
        <option value="1">1</option>
        <option value="2">2</option>
        <option value="3">3</option>
    </select>
	<button @click="add">加</button>
</template>
<script lang='ts' setup>
	import {ref} from "vue"
    let sum = ref(1)
    let n =ref(1)
    function add(){
        sum.value += n.value
    }
</script>
```

在main.ts中

```typescript
import {createPinia} from 'pinia'//引入pinia

const pinia = createPinia()//在创建app之后创建pinia
app.use(pinia)//安装pinia
```

## 存储数据

在count.ts中

```typescript
import {defineStore} from 'pinia'

export default const useCsountStore =defineStore('count',{
    state(){//真正存储数据的地方
        return {
            sum:1
        }
    }
})
```

## 读取数据

在Count.vue中

```vue
<template>
	<h2>当前求和为：{{countStore.sum}}</h2>
	<select v-model.number="n">
        <option value="1">1</option>
        <option value="2">2</option>
        <option value="3">3</option>
    </select>
	<button @click="add">加</button>
</template>
<script lang='ts' setup>
	import {ref} from "vue"
    import {useCsountStore} from '@/store/count'
    const countStore = useCsountStore()//是个reactive对象
    let n =ref(1)
    function add(){
        sum.value += n.value
    }
</script>
```

## 修改数据

```vue
<template>
	<h2>当前求和为：{{countStore.sum}}</h2>
	<select v-model.number="n">
        <option value="1">1</option>
        <option value="2">2</option>
        <option value="3">3</option>
    </select>
	<button @click="add">加</button>
</template>
<script lang='ts' setup>
	import {ref} from "vue"
    import {useCsountStore} from '@/store/count'
    const countStore = useCsountStore()//是个reactive对象
    let n =ref(1)
    function add(){
        countStore.sum+=1//第一种
        countStore.$patch({
            sum:888
        })//第二种
        countStore.increment(n.value)//第三种
    }
</script>
```

第三种方法需要在count.ts中加入

```typescript
import {defineStore} from 'pinia'

export default const useCsountStore =defineStore('count',{
    actions:{
        increment(value:number){
            this.sum+=value
        }
    }
    state(){//真正存储数据的地方
        return {
            sum:1
        }
    }
})
```

## storeToRefs方法

用于解构store中的响应式对象数据，不会将方法变成响应式

```vue
<template>
	<h2>当前求和为：{{sum}}</h2>
	<select v-model.number="n">
        <option value="1">1</option>
        <option value="2">2</option>
        <option value="3">3</option>
    </select>
	<button @click="add">加</button>
</template>
<script lang='ts' setup>
	import {ref} from "vue"
    import {useCsountStore} from '@/store/count'
    import {storeToRefs} from 'pinia'
    const countStore = useCsountStore()//是个reactive对象
    const {sum} = storeToRefs(countStore)
    let n =ref(1)
    function add(){
        countStore.sum+=1
    }
</script>
```

## getters配置项

```typescript
import {defineStore} from 'pinia'

export default const useCsountStore =defineStore('count',{
    actions:{
        increment(value:number){
            this.sum+=value
        }
    }
    state(){//真正存储数据的地方
        return {
            sum:1
        }
    },
    getters:{
        bigSum:state => state.sum*100//第一种写法
        bigSum(){
            return this.sum*100
        }//第二种写法
    }
})
```

## $subscribe使用

监视store数据变化，用法和watch类似

```vue
<template>
	<h2>当前求和为：{{sum}}</h2>
	<select v-model.number="n">
        <option value="1">1</option>
        <option value="2">2</option>
        <option value="3">3</option>
    </select>
	<button @click="add">加</button>
</template>
<script lang='ts' setup>
	import {ref} from "vue"
    import {useCsountStore} from '@/store/count'
    import {storeToRefs} from 'pinia'
    const countStore = useCsountStore()//是个reactive对象
    const {sum} = storeToRefs(countStore)
    let n =ref(1)
    countStore.$subscribe((mutate,state)=>{
      	...
})//mutate是新值,state是旧值
    function add(){
        countStore.sum+=1
    }
</script>
```

## store组合式写法

```typescript
import {defineStore} from 'pinia'
import {ref} from 'vue'
export default const useCsountStore =defineStore('count',()=>{
	let sum = ref(1)
    function increment(value:number){
            sum+=value
        }
    return {sum,increment}
})
```



# $event

`$event`对于原生事件，就是表示事件对象。可以.target

对于自定义事件，就是表示触发事件时，所传递的数据。不可以.target

# 组件通信

## props

用于父组件和之间传递数据子组件

父组件

```vue
<template>
	<div class="father">
        <h4>汽车:{{car}}</h4>
        <Child :car="car" :sendToy="getToy"/>
    </div>
</template>
<script lang='ts' setup name = "Father">
	import Child from './Child.vue'
    import {ref} from 'vue'
    let car = ref('奔驰')
    function getToy(value:string){
        console.log(value)
    }
</script>
```

子组件

```vue
<template>
	<div class="child">
        <h4>父汽车:{{car}}</h4>
        <h4>玩具:{{toy}}</h4>
        <button @click="sendToy(toy)"></button>//调用sendToy
    </div>
</template>
<script lang='ts' setup name="Child">
    import {ref} from 'vue'
    let toy = ref('猫')
    defineProps(['car','sendToy'])//声明接收props
</script>
```

## 自定义事件

专门用于子传父



父组件

```vue
<template>
	<div class="father">
        <h4>{{str}}</h4>
        <button @click="test(6,7,$event)"></button>
        <Child @xxx="xxx1"/>//xxx表示自定义名字，给子组件Child绑定事件
    </div>
</template>
<script lang='ts' setup name = "Father">
	import Child from './Child.vue'
    import {ref} from 'vue'
    let str = ref('你好')
    function test(a:number,b:number,c:Event){
        ...
    }
    function xxx1(value:number){
        console.log('xxx',value)
    }
</script>
```

子组件

```vue
<template>
	<div class="child">
        <h4>玩具:{{toy}}</h4>
        <button @click="emit('xxx',666)">触发事件</button>
    </div>
</template>
<script lang='ts' setup name="Child">
    import {ref} from 'vue'
    let toy = ref('猫')
    const emit = defineEmits(['xxx'])//声明事件
</script>
```

## mitt

可以实现任意组件通信

接收数据的：提前绑定好事件

提供数据的：在合适的时候触发事件

先使用`npm i mitt`安装mitt

在src目录下创建utils文件夹，在此文件夹中创建emitter.ts文件

```typescript
import mitt from 'mitt'
const emitter = mitt()
emitter.on('test1',()=>{//绑定事件
	...
})
setInterval(()=>{
    emitter.emit('test1')//触发事件
},1000)
setTimeout(()=>{
    emitter.off('test1')//解绑事件
    emitter.all.clear()//一键解绑
},3000)
export default emitter
```



组件中使用

组件1

```vue
<template>
	<div class="child1">
        <h4>玩具：{{toy}}</h4>
        <button @click="emitter.emit(send-toy,toy)"></button>
    </div>
</template>
<script lang='ts' setup name="Child1">
    import {ref} from 'vue'
    import emitter from '@/utils/emitter'
    let toy = ref('猫')

</script>
```

组件2

```vue
<template>
	<div class="child2">
        <h4>电脑：{{computer}}</h4>
        <h4>别人的玩具：{{toy}}</h4>
    </div>
</template>
<script lang='ts' setup name="Child2">
    import {ref,onUnmounted} from 'vue'
    import emitter from '@/utils/emitter'
    let computer = ref('苹果')
    let toy =ref('')
	emitter.on('send-toy',(value:string)=>{//收数据的那方绑定事件
        toy.value = value
    })
    onUnmount(()=>{//在组件卸载时解绑事件
		emitter.off('send-off')
    })
</script>
```

## v-model

主要用于UI组件库

父组件

```vue
<template>
	<div class="father">
        //<input type="text" v-model="username">用于html标签上
        //<XxxInput :modelValue="username" @update:modelValue ="username = $event"/>用于组件标签的源码
        <XxxInput v-model="username"/>//用于组件标签的写法
    </div>
</template>
<script lang='ts' setup name = "Father">
    import {ref} from 'vue'
    import Xxxinput from './Xxxinput.vue'
	let username =ref('张三')
</script>
```



Xxxinput组件源码

```vue
<template>
	<input type="test" :value="modelValue" @input="emit('update:modelValue',(<HTMLInputElemnet>$event.target).value)">
</template>
<script lang='ts' setup name = "XxxInput">
    defineProps(['modelValue'])
    const emit = defineEmits(['update:modelValue'])
</script>
```

## $attrs

用于实现当前组件的父组件，向当前组件的子组件通信，也就是爷和孙通信

父组件

```vue
<template>
	<div class="father">
		<Child :a="a" :b="b" :c="c" :d="d" :updateA="updateA"/>
    </div>
</template>
<script lang='ts' setup name = "Father">
    import {ref} from 'vue'
	import Child from './Child.vue'
	let a =ref(1)
    let b =ref(2)
    let c =ref(3)
    let d =ref(4)
     function updateA(value:number){
         a.value += value
     }
</script>
```

子组件

```vue
<template>
	<div class="child">
        <h4>{{a}}</h4>
        <h4>{{$attrs}}</h4>
		<GrandChild v-bind="$attrs"/>
    </div>
</template>
<script lang='ts' setup name="Child">
    import {ref} from 'vue'
	import GrandChild from './GrandChild.vue'
	defineProps(['a'])//只接收a，a就到props里，bcd在attrs中
</script>
```

孙组件

```vue
<template>
	<div class="grandchild">
        <button @click="updateA(6)"></button>
    </div>
</template>
<script lang='ts' setup name="GrandChild">
    import {ref} from 'vue'
	defineProps(['b','c','d','updateA'])
</script>
```

## \$refs \$parent

\$refs用于父传子

\$parent用于子传父

父组件

```vue
<template>
	<div class="father">
        <button @click="changeToy"></button>
        <button @click="getAllChild($refs)"></button>
		<Child1 ref="c1"/>
        <Child12 ref="c2"/>
    </div>
</template>
<script lang='ts' setup name = "Father">
    import {ref} from 'vue'
	import Child1 from './Child1.vue'
	import Child2 from './Child2.vue'
    let c1 = ref()
    let c2 = ref()
    let house = ref(4)
    function changeToy(){
        c1.value.toy = '狗'
    }
    function getAllChild(refs:object//或者使用any){//存的是个对象
        for(let key in refs){
            refs[key].book += 3
		}
    }
    //交出数据
    defineExpose({house})
</script>
```

子组件1

```vue
<template>
	<div class="child1">
        <button @click="minusHouse($parent)"></button>
    </div>
</template>
<script lang='ts' setup name="Child1">
    import {ref} from 'vue'
    let toy = ref('猫')
    let book = ref(3)
    function minusHouse(parent:any){
        parent.house -=1
    }
    //交出数据
    defineExpose({toy,book})
</script>
```

子组件2

```vue
<template>
	<div class="child2">
    </div>
</template>
<script lang='ts' setup name="Child2">
    import {ref} from 'vue'
    let computer = ref('苹果')
    let book = ref(6)
</script>
```

## provide-inject

用于爷和孙组件通信，不需要父组件

父组件

```vue
<template>
	<div class="father">
		<Child/>
    </div>
</template>
<script lang='ts' setup name = "Father">
    import {ref,reactive,provide} from 'vue'
	import Child from './Child.vue'
    let money = ref(100)
	let car = reactive({
        brand:'奔驰',
        price:100
    })
    function updateMoney(value:number){
        money.value -=value
    }
    provide('moneyContext',{money,updateMoney)//向全部后代提供数据，前面是名字，后面是数据
    provide('car',car)
</script>
```

子组件

```vue
<template>
	<div class="child">
		<GrandChild/>
    </div>
</template>
<script lang='ts' setup name="Child">
    import {ref} from 'vue'
	import GrandChild from './GrandChild.vue'
</script>
```

孙组件

```vue
<template>
	<div class="grandchild">
        <h4>{{money}}</h4>
        <h4>一辆{{car}},价值{{}}万</h4>
        <button @click="updateMoney(6)"></button>
    </div>
</template>
<script lang='ts' setup name="GrandChild">
    import {ref,inject} from 'vue'
    let {money,updateMoney} = inject('moneyContext',{money:0,updateMoney(x:number)=>{}})//可以加上默认值
    let car = inject('car',{brand:'未知',price:0})
</script>
```

## pinia

## slot

插槽

### 默认插槽

原本组件间夹的普通标签不会生效，使用插槽后能让它生效

父组件

```vue
<template>
	<div class="father">
        <Category title="xxx">
            <ul>
                <li v-for="g in games" :key="g.id">{{g.name}}</li>
    		</ul>
    	</Category>
    </div>
</template>
<script lang='ts' setup name = "Father">
	import Category from './Category.vue'
    import {ref,inject} from 'vue'
    let games = reactive([{id:'01',name:'lol'},
                        {id:'02',name:'cs'}
                        ])
</script>
```

子组件

```vue
<template>
	<div class="category">
        <h2>{{title}}</h2>
        <slot>默认内容</slot>
    </div>
</template>
<script lang='ts' setup name="Category">
    defineProps(['title'])
</script>
```

### 具名插槽

v-slot:可以简写为#

父组件

```vue
<template>
	<div class="father">
        <Category>
            <template v-slot:s2>
                <ul>
                	<li v-for="g in games" :key="g.id">{{g.name}}</li>
    			</ul>
			</template>
			<template v-slot:s1>
				<h2>xxx</h2>
			</template>
    	</Category>
    </div>
</template>
<script lang='ts' setup name = "Father">
	import Category from './Category.vue'
    import {ref,inject} from 'vue'
    let games = reactive([{id:'01',name:'lol'},
                        {id:'02',name:'cs'}
                        ])
</script>
```

子组件

```vue
<template>
	<div class="category">
        <slot name="s1">默认内容</slot>
        <slot name="s2">默认内容</slot>
    </div>
</template>
<script lang='ts' setup name="Category">
</script>
```

### 作用域插槽

父组件

```vue
<template>
	<div class="father">
        <Game>
            <template v-slot="params">//a是个接收所有传入数据的对象
                <ul>
            		<li v-for="g in params.games",:key="g.id">{{g.name}}</li>
    			</ul>
			</template>
    	</Game>
    </div>
</template>
<script lang='ts' setup name = "Father">
    import Game from './Game.vue'
    import {ref,inject} from 'vue'
    let games = reactive([{id:'01',name:'lol'},
                        {id:'02',name:'cs'}
                        ])
</script>
```

子组件

```vue
<template>
	<div class="game">
        <slot :games="games"></slot>
    </div>
</template>
<script lang='ts' setup name="Game">
    import {ref,inject} from 'vue'
    let games = reactive([{id:'01',name:'lol'},
                        {id:'02',name:'cs'}
                        ])
</script>
```

