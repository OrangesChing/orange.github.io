---
title: Vue入门
date: 2020-08-19 15:45:41
tags:
  - 前端技术
categories:
  - 前端框架
  - Vue
---

# Vue.js 目录结构

在 IDE 中打开目录，结构如下所示：

![Vue目录结构](Vue入门/Vue目录结构.jpg)


| 目录/文件    | 说明                                                         |
| :----------- | :----------------------------------------------------------- |
| build        | 项目构建(webpack)相关代码                                    |
| config       | 配置目录，包括端口号等。我们初学可以使用默认的。             |
| node_modules | npm 加载的项目依赖模块                                       |
| src          | 这里是我们要开发的目录，基本上要做的事情都在这个目录里。里面包含了几个目录及文件：assets: 放置一些图片，如logo等。<br />components: 目录里面放了一个组件文件，可以不用。<br />App.vue: 项目入口文件，我们也可以直接将组件写这里，而不使用 components 目录。<br />main.js: 项目的核心文件。 |
| static       | 静态资源目录，如图片、字体等。                               |
| test         | 初始测试目录，可删除                                         |
| .xxxx文件    | 这些是一些配置文件，包括语法配置，git配置等。                |
| index.html   | 首页入口文件，你可以添加一些 meta 信息或统计代码啥的。       |
| package.json | 项目配置文件。                                               |
| README.md    | 项目的说明文档，markdown 格式                                |

# Vue构造器

## Vue构造器初识

**每个 Vue 应用都需要通过实例化 Vue 来实现。**

语法格式如下：

```vue
var vm = new Vue({
  // 选项
})
```

 Vue 构造器简单使用例子：

```vue
<div id="vue_det">    
    <h1>name : { {name} }</h1>     <!-- { { } } 用于输出对象属性和函数返回值 -->
    <h1>message : { {msg} }</h1>     
    <h1>{ {details()} }</h1> 
</div> 
<script type="text/javascript">     
    var vm = new Vue({         
        el: '#vue_det',       // DOM 元素中的 id，说明接下来的改动全部在指定div内，div外部不受影响
        data: {               // data用于定义属性，实例中有两个属性分别为：name、msg。 
            msg: "HelloWorld",             
            name: "orange"      
        },         
        methods: {            // methods用于定义的函数，可以通过 return 来返回函数值。
            details: function() {                 
                return  this.name + " Hello！";             
            }         
        }     
    }) 
</script>

<!--另一种data赋值法展示出来的特性:无论改哪个结果一样-->
<script type="text/javascript"> 
    var data = { 
        msg: "HelloWorld",             
        name: "orange"
    } 
    var vm = new Vue({
        el: '#vue_det',    
        data: data    // 可理解为vm.data指向了data内存，所以下面无论修改vm.data还是data结果都一样
    }) // 它们引用相同的对象！ 
    document.write(vm.msg === data.msg) // true 
    document.write("<br>") // 设置属性也会影响到原始数据 
    vm.msg = "Hello" 
    document.write(data.site + "<br>") // Hello
    data.name = "leaf" 
    document.write(vm.name) // leaf
</script>
```

> 注意：
>
> 当一个 Vue 实例被创建时，它向 Vue 的响应式系统中加入了其 data 对象中能找到的所有的属性。当这些属性的值发生改变时，html 视图将也会产生相应的变化

## Vue构造器属性

这些属性可使用`vm.$属性名访问`

| 属性名       | 作用                                | 备注                               |
| ------------ | ----------------------------------- | ---------------------------------- |
| `el`         | 申明vue控制范围，为实例提供挂在元素 |                                    |
| `data`       | 模板数据                            |                                    |
| `components` | Vue实例配置局部注册组件             |                                    |
| `methods`    | 实例方法                            |                                    |
| `watch`      | 侦听属性，监测变化                  |                                    |
| `computed`   | 计算属性，也会监测变化，但有返回值  | 当数据发生变化时才调用，可提升性能 |
| `filters`    | 过滤器                              |                                    |
| `mouted`     | 钩子函数                            |                                    |
| `render`     | 渲染函数，创建虚拟DOM               |                                    |

```jsx
new Vue({
    el: "#app" //为实例提供挂载元素，该元素下被vue控制
    data: {
        firstName: 'Foo',
        lastName: 'Bar',
        fullName: 'Foo Bar' //初始化实例属性
    },
    props: ["content"], //父组件向子组件传值
    methods: {
        getHandle(){
            //该属性中申明方法
        }
    },
    mounted: function(){
        // 钩子函数
    },
    computed: {
        //计算属性，当数据发生变化时才调用，否则直接调用缓存。提升性能
        fullName: function () {
            return this.firstName + ' ' + this.lastName
        }
    },
    watch: {
        //监测变化。
        firstName: function (val) {
            this.fullName = val + ' ' + this.lastName
        },
        lastName: function (val) {
            this.fullName = this.firstName + ' ' + val
        }
    }
});
```

### 属性操作

Vue 不允许在已经创建的实例上动态添加新的根级响应式属性，如果我们需要在运行过程中实现属性的添加或删除，则可以使用全局 Vue.set` 和 `Vue.delete` 方法

```js
Vue.set( target, key, value )
Vue.delete( target, key )
```

### 计算属性

 methods 同 computed效果上是一样的，但 **computed 是基于它的依赖缓存，只有相关依赖发生改变时才会重新取值**。而使用 methods ，在重新渲染的时候，函数总会重新调用执行。所以**使用 computed 性能会更好**

```vue
<div id="app">
  <p>原始字符串: { { message } }</p>
  <p>计算后反转字符串: { { reversedMessage } }</p>
</div>
 
<script>
var vm = new Vue({
  el: '#app',
  data: {
    message: 'Runoob!'
  },
  computed: {
    // 计算属性的 getter
    reversedMessage: function () {
      // `this` 指向 vm 实例
      return this.message.split('').reverse().join('')
    }
  }
  methods: {     // methods 与 computed效果一样，但是computed只有数据发生变化时才会执行
    reversedMessage2: function () {
      return this.message.split('').reverse().join('')
    }
  }
})
</script>
```

getter与setter方法

```js
var vm = new Vue({
  el: '#app',
  data: {
    name: 'Google',
    url: 'http://www.google.com'
  },
  computed: {
    site: {
      // getter 如果只要getter方法时可直接写方法体，不要get
      get: function () {
        return this.name + ' ' + this.url
      },
      // setter
      set: function (newValue) {
        var names = newValue.split(' ')
        this.name = names[0]
        this.url = names[names.length - 1]
      }
    }
  }
})
```

### 监听属性

```vue
<div id = "app">
    <p style = "font-size:25px;">计数器: { { counter } }</p>
    <button @click = "counter++" style = "font-size:25px;">点我</button>
</div>
<script type = "text/javascript">
    var vm = new Vue({
        el: '#app',
        data: {
            counter: 1
        }
    });
    vm.$watch('counter', function(nval, oval) {
        alert('计数器值的变化 :' + oval + ' 变为 ' + nval + '!');
    });
</script>
```



# Vue模板

## 指令

**指令是带有 v- 前缀的特殊属性**

| 类型             | 语法                                                         | 示例                                                         |
| ---------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 插文本值         | `{ {message} }`                                              | `<p>{ { message } }</p>`                                     |
| HTML指令值       | `v-html`                                                     | `<div v-html="msg"></div> `                                  |
| 属性指令         | `v-bind`（可缩写为`:class`）                                 | `<div v-bind:class="{'class1': use}"></div>`                 |
| 条件指令         | `v-if`<br />`v-else`<br />`v-else-if`<br />`v-show`根据条件展示 | `<div v-if="type === 'A'"> A </div>`<br />`<div v-else-if="type === 'B'"> B </div>`  <br /> `<div v-else> C </div>`<br />`<h1 v-show="ok"> Hello! </h1>` |
| 循环指令         | `v-for`                                                      | `<li v-for="site in sites"> { { site.name } } </li>`<br />`<li v-for="(v, k) in object"> { { k } } : { { v } } </li>`<br />`<li v-for="(v, k, i) in object"> { { i } } . { { k } } : { { v} } </li>` |
| 用户输入双向绑定 | `v-model`双向绑定                                            | `<input v-model="message">`                                  |
| 事件处理器指令   | `v-on`                                                       | `<button v-on:click="reverseMessage">反转</button>`          |
| 修饰符           | 以半角句号`.`指明的特殊后缀                                  | `<form v-on:submit.prevent="onSubmit"></form>`               |

### 文本 { { } }

```vue
<div id="app">   
	<p>{ { message } }</p> 
</div>
```

### HTML v-html

```vue
<div id="app">
    <div v-html="message"></div>
</div>
    
<script>
new Vue({
  el: '#app',
  data: {
    message: '<h1>hello</h1>'
  }
})
</script>
```

### 属性 V-bind

```vue
<div id="app">   
    <label for="r1">修改颜色</label>
    <!--单值形式-->
    <div v-bind:class="{'class1': use }">      
    </div> 
    <!--数组形式-->
    <div v-bind:class="[activeClass, errorClass]">		</div>
    <!--内联样式绑定-->
    <div v-bind:style="[baseStyles, overridingStyles]">	   </div>
</div>      
<script> 
    new Vue({     
        el: '#app',   
        data:{       
            use: false   
        } 
    }); 
</script>
```

### 事件处理指令 v-on

```vue
<div id="app">
  <button v-on:click="counter += 1">增加 1</button>
  <p>这个按钮被点击了 { { counter } } 次。</p>
</div>
 
<script>
new Vue({
  el: '#app',
  data: {
    counter: 0
  }
})
</script>
```

#### 事件修饰符

Vue.js 为 v-on 提供了事件修饰符来处理 DOM 事件细节，通过由点(`.`)表示的指令后缀来调用修饰符。

- `.stop`
- `.prevent`
- `.capture`
- `.self`
- `.once`

```html
<!-- 阻止单击事件冒泡 -->
<a v-on:click.stop="doThis"></a>
<!-- 提交事件不再重载页面 -->
<form v-on:submit.prevent="onSubmit"></form>
<!-- 修饰符可以串联  -->
<a v-on:click.stop.prevent="doThat"></a>
<!-- 只有修饰符 -->
<form v-on:submit.prevent></form>
<!-- 添加事件侦听器时使用事件捕获模式 -->
<div v-on:click.capture="doThis">...</div>
<!-- 只当事件在该元素本身（而不是子元素）触发时触发回调 -->
<div v-on:click.self="doThat">...</div>
<!-- click 事件只能点击一次，2.1.4版本新增 -->
<a v-on:click.once="doThis"></a>
```

#### 按钮修饰符

Vue 允许为 v-on 在监听键盘事件时添加按键修饰符

- `.enter`
- `.tab`
- `.delete` (捕获 "删除" 和 "退格" 键)

- `.esc`

- `.space`

- `.up`
- `.down`

- `.left`

- `.right`

- `.ctrl`

- `.alt`

- `.shift`

- `.meta`

```vue
<!-- 同上 -->
<input v-on:keyup.enter="submit">
<!-- 缩写语法 -->
<input @keyup.enter="submit">
```

### 表单双向绑定 v-model

![双向绑定](Vue入门/双向绑定.png)

以下是对不同DOM元素的绑定

```vue
<div id="app">
    <!-- 文本输入框用单个变量绑定 -->
    <input v-model="message" placeholder="编辑我……">
    <p>消息是: { { message } }</p>
    <textarea v-model="message2" placeholder="多行文本输入……"></textarea>
    <p>{ { message2 } }</p>

    <!-- 单个复选框用单个变量绑定 -->
    <input type="checkbox" id="checkbox" v-model="checked">
    <label>{ { checked } }</label>

    <!-- 多个复选框用数组绑定 -->
    <input type="checkbox" id="baidu" value="Baidu" v-model="checkedNames">
    <label>Runoob</label>
    <input type="checkbox" id="google" value="Google" v-model="checkedNames">
    <label>Google</label>
    <input type="checkbox" id="taobao" value="Taobao" v-model="checkedNames">
    <label>taobao</label>
    <span>{ { checkedNames } }</span>
    
    <!--单选框用单个变量绑定-->
    <input type="radio" id="baidu" value="Baidu" v-model="picked">
  	<label for="baidu">baidu</label>
  	<input type="radio" id="google" value="Google" v-model="picked">
  	<label for="google">Google</label>
  	<span>{ { picked } }</span>
    
    <!--select用单个变量绑定-->
    <select v-model="selected" name="fruit">
        <option value="">选择一个网站</option>
        <option value="www.baidu.com">Baidu</option>
        <option value="www.google.com">Google</option>
  	</select>
    <span>{ { selected } }</span>
</div>
<script>
    new Vue({
      el: '#app',
      data: {
        message: 'hello',
        message2: 'orange'
        checked : false,
        checkedNames: []
        picked : 'baidu'
        selected: '' 
      }
    })
</script>
```

#### 修饰符

在默认情况下， v-model 在 input 事件中同步输入框的值与数据（即边input框输入数据发生改变时就会触发）

**`.lazy`**

添加`lazy`修饰符 后 ，将把 `oninput` 事件改成 `onchange` 事件（即全部输入完成焦点不在input框上时触发）

```html
<!-- 在 "change" 而不是 "input" 事件中更新 -->
<input v-model.lazy="msg" >
```

**`.number`**

如果想自动将用户的输入值转为 Number 类型（如果原值的转换结果为 NaN 则返回原值），可以添加一个修饰符 `number` 给 `v-model`来处理输入值：

```vue
<input v-model.number="age" type="number"><!--输入123aaa age的值为123； 输入为aaa123 age的值为123aaa--->
```

> 需要注意的是，如果输入的第一个字是字符串，那`number`修饰符就不会生效，会返回原值
> 输入的第一个只能是数字或者小数点或者是正负号

**`.trim`**

如果要自动过滤用户输入的首尾空格，可以添加 trim 修饰符到 v-model 上过滤输入

```vue
<input v-model.trim="msg">
```

## 表达式

Vue提供完全的 JS 表达式支持

```vue
<div id="app">     
    { {5+5} }<br>     
    { { ok ? 'YES' : 'NO' } }<br>     
    { { message.split('').reverse().join('') } }     
    <div v-bind:id="'list-' + id">helloWorld</div> 
</div>      
<script> 
    new Vue({   
        el: '#app',   
        data: {     
            ok: true,     
            message: 'hello',     
            id : 1   
        } 
    }) 
</script>
```

## 过滤器

过滤器是 JavaScript 函数，可自定义，被用作一些常见的文本格式化。由"管道符"指示, 用法如下：

```vue
<!-- 过滤器函数接受表达式的值作为第一个参数 -->
{ { message | capitalize } }
<!-- 可串联 -->
{ { message | filterA | filterB } }
<!--可接受参数,message 是第一个参数，字符串 'arg1' 将传给过滤器作为第二个参数， arg2 表达式的值将被求值然后传给过滤器作为第三个参数-->
{ { message | filterA('arg1', arg2) } }
<!-- 在 v-bind 指令中 -->
<div v-bind:id="rawId | formatId"></div>
```

以下实例对输入的字符串第一个字母转为大写：

```vue
<div id="app">   
    { { message | capitalize } } 
</div>      
<script> 
    new Vue({
        el: '#app',   
        data: {     
            message: 'hello'   
        },   
        filters: {     
            capitalize: function (value) {       
                if (!value) 
                    return ''       
                value = value.toString()       
                return value.charAt(0).toUpperCase() + value.slice(1)     
            }   
        } 
    }) 
</script>
```

# Vue 组件

## 组件是什么

组件封装了HTML代码，可扩展 HTML 元素。

组件系统让我们可以用独立可复用的小组件来构建大型应用，几乎任意类型的应用的界面都可以抽象为一个组件树

![组件示意图](Vue入门/组件示意图.png)



## 定义组件

全局组件使用`Vue.component`创建，可以用在任何新创建的vue根实例(`new Vue`)中。全局组件必须写在根实例的前面才会生效。局部组件可以减少无谓的JS下载，但局部组件默认不能在子组件中调用

### 全局组件

```vue
<div id="app">
    <orange></orange>  <!--调用组件-->
</div>
 
<script>
// 注册
Vue.component('orange', {
  template: '<h1>自定义组件!</h1>'
})
// 创建根实例
new Vue({
  el: '#app'
})
</script>
```

### 局部组件

```vue
<div id="app">
    <orange></orange>
</div>
 
<script>
var orangeComponent = {
    template: '<h1>自定义组件!</h1>'
}    
    
// 创建根实例
new Vue({
  el: '#app',
  components: {
    // <orange> 将只在父模板可用
    'orange': orangeComponent
  }
})
</script>
```

## <h2 id="组件之间的数据传递">组件之间的数据传递</h2>
### 父组件->子组件   prop
#### prop

```vue
<div id="app">     
    <child message="hello!"></child> 
</div>   
<script> 
    // 注册 
    Vue.component(
        'child', {   
            // 声明 props   
            props: ['message'],   // 同样也可以在 vm 实例中像 "this.message" 这样使用   
            template: '<span>{ { message } }</span>' 
    }) 
    // 创建根实例 
    new Vue({   el: '#app' }) 
</script>
```

#### 动态 Prop

可以用 v-bind 动态绑定 props 的值到父组件的数据中。每当父组件的数据变化时，该变化也会传导给子组件

```vue
<div id="app">
    <div>
      <input v-model="parentMsg">
      <br>
      <child v-bind:message="parentMsg"></child>
    </div>
</div>
 
<script>
// 注册
Vue.component('child', {
  // 声明 props
  props: ['message'],
  // 同样也可以在 vm 实例中像 "this.message" 这样使用
  template: '<span>{ { message } }</span>'
})
// 创建根实例
new Vue({
  el: '#app',
  data: {
    parentMsg: '父组件内容'
  }
})
</script>
```

#### prop的验证

```js
Vue.component('my-component', {
  props: {
    // 基础的类型检查 (`null` 和 `undefined` 会通过任何类型验证)
    propA: Number,
    // 多个可能的类型
    propB: [String, Number],
    // 必填的字符串
    propC: {
      type: String,
      required: true
    },
    // 带有默认值的数字
    propD: {
      type: Number,
      default: 100
    },
    // 带有默认值的对象
    propE: {
      type: Object,
      // 对象或数组默认值必须从一个工厂函数获取
      default: function () {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propF: {
      validator: function (value) {
        // 这个值必须匹配下列字符串中的一个
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    }
  }
})
```

### 子组件->父组件  自定义事件

父组件是使用 props 传递数据给子组件，但如果子组件要把数据传递回去，就需要使用自定义事件！

我们可以使用 v-on 绑定自定义事件, 每个 Vue 实例都实现了事件接口(Events interface)，即：

- 使用 `$on(eventName)` 监听事件
- 使用 `$emit(eventName)` 触发事件

另外，父组件可以在使用子组件的地方直接用 v-on 来监听子组件触发的事件。

```vue
<div id="app">
    <div id="counter-event-example">
      <p>{ { total } }</p>
      <button-counter v-on:increment="incrementTotal"></button-counter>
      <button-counter v-on:increment="incrementTotal"></button-counter>
    </div>
</div>
 
<script>
Vue.component('button-counter', {
  template: '<button v-on:click="incrementHandler">{ { counter } }</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    incrementHandler: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
})
new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
</script>
```

# Vue Ajax(axios)

## GET

```js
new Vue({
  el: '#app',
  data () {
    return {
      info: null
    }
  },
  mounted () {
    axios
      .get('https://www.runoob.com/try/ajax/json_demo.json')
      .then(response => (this.info = response))
      .catch(function (error) { // 请求失败处理
        console.log(error);
      });
  }
})
```

传递参数

```js
// 直接在 URL 上添加参数 ID=12345
axios.get('/user?ID=12345')
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });
 
// 也可以通过 params 设置参数：
axios.get('/user', {
    params: {
      ID: 12345
    }
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });
```

## POST

```js
new Vue({
  el: '#app',
  data () {
    return {
      info: null
    }
  },
  mounted () {
    axios
      .post('https://www.runoob.com/try/ajax/demo_axios_post.php')
      .then(response => (this.info = response))
      .catch(function (error) { // 请求失败处理
        console.log(error);
      });
  }
})
```

传递参数

```js
axios.post('/user', {
    firstName: 'Fred',        // 参数 firstName
    lastName: 'Flintstone'    // 参数 lastName
  })
  .then(function (response) {
    console.log(response);
  })
  .catch(function (error) {
    console.log(error);
  });
```

## 其他

```
axios.request(config)
axios.get(url[, config])
axios.delete(url[, config])
axios.head(url[, config])
axios.post(url[, data[, config]])
axios.put(url[, data[, config]])
axios.patch(url[, data[, config]])
```