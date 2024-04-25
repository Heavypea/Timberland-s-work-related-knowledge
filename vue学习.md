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

`})`

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

