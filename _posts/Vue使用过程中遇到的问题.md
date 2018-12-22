---
title: Vue使用过程中遇到的问题
abbrlink: 67b3
date: 2018-09-29 21:09:18
tags:
	- Vue
---




使用Vue的过程中碰到的各种错误，统计记下来

*引入的组件如果有一个组件出现了问题，会导致在最终渲染生成的html许多资源注入失败*

#### 到底何为数据驱动？

比如一个页面显示10个item，随着用户下滑，item应该随时更新，那么就可以有一个数组，不断地请求新数据，
当新的数据到达之后就更新，vue使用者只需要准确更新数据，而数据最后在视图层的展示则由vue去处理

这种情况下适合使用vuex。

既然如此，那么vuex store中应该存放随外界条件改变的`状态`，对于一些不会改变的变量则不适合用vuex store

我也不知道这样理解对不对

<!--more-->

对于频繁改变的数据存放到vuex store，那么对于不变的实例如果存放？

1. 父组件向子组件传递使用props

2. 子组件向上传递使用事件

   *子组件*<Post/>

   ```javascript
   this.$emit('someEvent')
   ```

   *父组件<Blog>*

   ```vue
   <Blog>
   <Post
   	@someEvent="handleSomeEvent"
   	/>
   </Blog>
   ```

3. 对于没有父子关系的组件：
   1. 如果数据只在初始化时发生改变，那么适合使用空`vue`来做`eventbus`
   2. 如果数据和视图进行了绑定，那么使用`Vuex`来追踪响应式变化



#### 父子组件的初始化顺序

使用props时，父子组件的初始化顺序到底是什么样的，如果子组件访问从父组件传递的数据时，父组件还未初始化，这样如何处理？

*之后添加*

###### 现在可以解答这个问题了

我在父子组件的不同生命周期上下了几个断点，最后得出的结论如下：

1. created由App.vue顶层 开始，逐步传递到最深的子组件
2. mounted从最深子组件开始，一直传递到顶层为止
3. nextTick由最深子组件开始，一直传递到顶层为止



父组件不创建就没有子组件的存在，没有父亲哪来儿子？

创建之后的DOM要挂载到浏览器DOM树上，外层DOM基于内层DOM，毕竟内层是基石（强行理解）

nextTick总感觉和冒泡一样，事件的传递由内层开始向外扩散

这里只测试了父子组件之间的关系，并没有测试兄弟组件之间的顺序



我在这里碰到了一个问题

如果子组件需要父组件(App.vue)的一个变量，但父组件中此变量的生成必须在`App.vue`挂载到DOM之后才能获得，也就是需要在DOM节点上展开自定义元素，这样子组件必须在父组件`mounted`之后才能获得这个变量，但`mounted`从内到外，`nextTick`也是从内到外，无论如何子组件的都在前面先进行



最后通过生命周期钩子和空`Vue`来做`EventBus`解决了这点

父组件App.vue

```vue
mounted (){
  this.ide = new IDE('DOMElement')//DOMElement必须在App.vue挂载完成之后才能获得
  eventBus.$emit('init',{myide:this.ide})
}
```



子组件FileTree.vue

```
created (){
  eventBus.$on('init',(data) =>{
    this.ide = data.myide
  })
}
```



--------------------------------------------------------------------------------------------------------------------------------------------------

#### 计算属性

关于`Vue`的计算属性，个人理解，`computed`相对于methods不同之处在于当`computed`中的`function`中依赖的外部变量变化时，这个函数会被调用
而`methods`中的函数由用户进行显式调用

正是由于`computed`的这个特性，所以可以将处理组件当中关联DOM的数据变量放到`computed`函数中，当变量由于外界条件变化而变化的时候，组件会重新渲染

#### 关于Vue project实例的渲染顺序
从main.js触发，进行初始化
```javascript
new Vue({
  el: '#app',
  components: {
    App
  },
  store,
  template: '<App/>'
})
```
由于引用了App组件，所以执行App组件下的代码
根据代码中import的顺序，从头开始import并执行代码

```javascript
import Vue from "Vue"
import ElementUI from "element-ui"
```
Vue中如果有import,那就跟进初始化
当Vue中全部的引用初始化完成之后开始进入Element的初始化

有这个问题是经常组件之间会相互引用数据，由于import顺序不对，经常会出现`undefined`的问题

也不确定这里理解对不对，不过到现在为止，这样理解确实是对的。



#### 如何获取已经注册到Vue的组件对象？

最近在使用`element-ui`组件库的`el-tree`组件，第一次用，想要调用这个组件下的一些方法，最终发现可以在官网实例中可以通过`this.$refs`调用。

```html
<el-tree
      ref="maintree"
      :data="files"
      :allow-drop="allowDrop"
      :allow-drag="allowDrag"
      draggable
      node-key="id"
      default-expand-all
      @node-drag-start="handleDragStart"
      @node-drag-enter="handleDragEnter"
      @node-drag-end="handleDragEnd"
      @node-drag-leave="handleDragLeave"
      @node-drop="handleDrop"
    />
  </div>
```

```javascript
methods:{
  getCurrentNode(){
    let currentNode = this.$refs.maintree.getCurrentNode()
    console.log(currentNode)
  }
}
```



`Vue`官方`api `给的`$refs`定义是

> An object of DOM elements and component instances, registered with `ref` attributes.

通过在组件上使用`ref`就可以给这个组件一个名字，这个名字还会被注册到Vue实例上，然后就可以使用组件下的方法



#### v-show 和v-if



碰到一个问题，想要根据`boolean`来条件显示一个DOM块，想当然的选了v-if

但是v-if只有当条件为真的时候才会第一次渲染DOM，所以我在v-if="false"的时候尝试获取其DOM元素，一直得到undefined，回头查了一下文档，发现文档说的很详细，这里我也看到过，但是由于没有实际应用，所以忽略了，等到出了问题，卡在这能有半天

> `v-if` 也是**惰性的**：如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。
>
> 相比之下，`v-show` 就简单得多——不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 进行切换。



一个极为相似的问题

[This.$refs return empty object](https://forum.vuejs.org/t/solved-this-refs-return-empty-object/33227)



#### computed属性

```javascript
data () {
  return {
    a:1,
    b:2
  }  
},
computed:{
  sum:{
  	get() {
    	return this.a + this.b  
  	},
  	set(value){
      
  	}
  }
}
```

存储在data中的数据和vuex中的数据是响应式的，但是如果我们实际需要的是`a + b`的和，那么可以用计算属性，这样我们可以直接使用sum来得到结果，当a,b 值发生变化，sum值也会跟着变化。

**但响应式变化的前提是引用了响应式的变量**



正如官方给的例子

```javascript
computed:{
  now(){
    return Date.now()
  }
}
```

因为`Date.now()`并没有响应式依赖，所以值一直不变化。不能将需要响应式变化的值直接存放到`computed`属性当中。

类似的还有`Vuex`的`Getter`，如果想要从`state`中**派生**出一些状态，如果数组当中小于10的元素的个数，那么就可以使用`Getter`

```vue
getters:{
	count: state => {
		state.array.filter(o => o < 10).length	
	} 
}
```



#### 做个记录

`Vue`的`data`，生命周期钩子都是函数

```vue
data() {
	return{
    
	}
},
created(){

},
mounted(){

},
updated() {

}
```



#### 动态组件的加载和数据的动态传递

今天碰到了一个需求，想要实现一个右键菜单，这个菜单需要根据右击的条目不同来显示不同的选项

所以想到了使用动态组件

下面的都是我在`StackOverflow`上查到了，然后更改使用的

[向动态组件传递动态参数](https://stackoverflow.com/questions/43658481/passing-props-dynamically-to-dynamic-component-in-vuejs)

###### 父组件

*模板部分*

```html
<ContextMenu
	:is="currentContextMenu"
	:items="currentProperties"
  @delete="delete"
	ref="context_menu"
```



*Javascript部分*

```javascript
import ContextMenu from './ContextMenu.vue'
export default{
	components:{
		'fileContextMenu': ContextMenu,
    'folderContextMenu': ContextMenu
	}，
  data (){
  	return {
  		currentContextMenu:'fileContextMenu'
		}
	},
	computed:{
		currentProperties (){
  		if (this.currentContextMenu === 'file'){//如果文件类型是file，那么右键就显示删除文件
  			return [
  				{
  					"name":"DeleteFile",
  					"action":"delete"
					}
  			]
			}else{//是文件夹就显示删除文件夹
        return[
          {
            "name":"DeleteFolder",
            "action":"delete"
          }
        ]
			}
		}
	}
}
```



当在条目之上**右键**点击时回调会被调用,详情参照`element-ui`的`tree`组件和其`@node-contextmenu`事件

每个Node的格式（文件类型为file或者folder其中一个，ID为node-key）

```json
{
  detail:{
    type:'file' | 'folder'
  },
  id:number,
}
```



```javascript
handleContextMenu(event,o,node.vueComponent){
  if(node.data.detail.type ==='file'){
    this.currentContextMenu = 'fileContextMenu'
  }else{
		this.currentContextMenu = 'folderContextMenu' 
  }
  this.$refs.context_menu.openMenu()
}
```

这样我们只需要更改`:is`指向的组件就可以了

这里会引出一个问题，也是我试了好多次才解决的

当`this.currentContext`在`handleContextMenu`更改之后，`computed`中的`currentProperties`应该更改，

事实上确实计算属性更改了，但是顺序却不对，因为新的右键菜单必须在`currentProperties`更新之后才是正确的。右键点击文件夹之后，组件直接更新，但是我们之前如果点击的是文件，那么

```
组件更新成folder => 右键菜单被渲染 => computed.currentProperties计算属性重新计算。
```

所以显示是file的右键菜单。

最后解决办法是使用`nextTick`

```javascript
handleContextMenu(event,o,node.vueComponent){
  if(node.data.detail.type ==='file'){
    this.currentContextMenu = 'fileContextMenu'
  }else{
		this.currentContextMenu = 'folderContextMenu' 
  }
  this.$nextTick( () =>{
    this.$refs.context_menu.openMenu()
  })
}
```

来延迟一下菜单的打开

#### mapGetters使用

假如由如下`store`  `global.js`

```javascript
export default{
  namespaced: true,
  state: {
    ide: undefined,
    terminal: undefined,
    workspace: {
      filetree: {
        files: [
          {
            detail: {
              type: 'dir'
            },
            id: 1,
            label: 'source-codes',
            children: [
              {
                detail: {
                  type: 'file',
                  src: ''
                },
                id: 2,
                label: 'main.go'
              }
            ]
          }
        ]
      },
      ide: {
        lang: 'golang',
        suffix: '.go',
        theme: 'chrome',
        fontsize: 14
      }
    }
  },
  mutations: {
    setIDE (state, myide) {
      state.ide = myide
    },
    setWorkSpace (state, workspace) {
      state.workspace = workspace
    },
    getWorkSpace (state) {
      return state.workspace
    }
  },
  getters: {
    filesList: state => state.workspace.filetree.files
  }
}
```



以及`store`中的index.js

```javascript
import Vue from 'vue'
import Vuex from 'vuex'
import global from './modules/global'

Vue.use(Vuex)

const store = new Vuex.Store({
  // // strict:process.env.NODE_ENV !== 'production',
  // strict:true,
  modules: {
    global
  }
})

export default store

```

现在我们要映射`this.files => this.$store.global.filesList`

有如下两种写法

1. ```javascript
   ...mapGetters({
         files: 'global/filesList'
   }),
   ```

2. ```javascript
   ...mapGetters('global', ['filesList']),
   ```



因为`global.js`位于store子模块，所以要使用global作为模块名来引用

第一种方式重新给`filesList`了一个新的名称`files`

第二种可以用来批量映射

这是在`Chrome`里面调试的，变量直接显示的是`global/filesList`



如果想要映射`state`下的变量，可以这样

```javascript
...mapState({
  files:state => state.global.workspace.filetree.files
})
```

这样也可以（这样不是映射files,而是workspace）

```
...mapState('global',['workspace'])
```



但是我不明白同样都是`map`

为什么下面的写法就不能用

```javascript
...mapState({
  files:'global/workspace.filetree.files'
})
```

---------------------

之后写的，来解答上面的问题

```javascript
..mapState('global',['workspace.filetree.files'])
```

这种的函数签名显示的是`namspace,string[]`，也就是这种开头指定命名空间，后面则是映射的对象

最后会映射成`this.workspace.filetree.files`，这种方式映射不能更改名字

如果想要更改成当前组件中使用的名字要使用下面的方式





```javascript
...mapState({
	files: state => state.global.workspace.filetree.files  
})
```





[mapMutations](https://github.com/nuxt/nuxt.js/blob/dev/examples/vuex-store-modules/pages/todos.vue)

从上面的连接找到了

```javascript
 ...mapMutations({
      toggle: 'todos/toggle'
 })
```

也就是`mapMutations`也要加`namespace，`

总感觉`action`,`mutations`,`getters`都要指定`namespace`

-------------------------------------------------

之后添加的：

mutations, getters action确实有namespace，前提是在子模块使用了`namespaced:true`，我看`Vuex`文档唯独没有看`module`这部分，全部的问题都在这一章给解决了。

https://vuex.vuejs.org/zh/guide/modules.html 



而且`mutations`是函数，应该将`mapMutations`写到`methods:{}`里面，而不是`computed`里面



#### 2018-9-26添加

突然知道为什么不推荐使用`EventBus`了，跨组件传递使用事件确实方便，但是如果想要重构代码实在是麻烦，找半天也找不到这个事件是谁推送的，而且由于使用事件打乱了正常的数据传递，真的是牵一发而动全身



------------------------------

#### 如何在Javascript Class里面使用Vuex store实例



之前不会，现在明白了，虽然很简单，但还是决定记录下来

自己有一个类，因为功能独立于组件，所以把它单独写到了`js`文件当中。

大体就是下面这样:

```javascript
import * as Ace from 'ace-builds'
import 'ace-builds/src-noconflict/ext-language_tools'
import store from './store'
import Vue from 'vue'
import eventBus from './eventbus.js'
export default class IDE {
  constructor (id) {
    this.$store = store
    Ace.config.set('basePath', 'static/js/ace')
    this.aceEditor = Ace.edit(id)
    this.aceSession = this.aceEditor.session
    this.aceSession.setMode('ace/mode/golang')
    this.$http = Vue.prototype.$http
    this.$terminal = store.state.global.terminal
  }
  resize () {
    this.aceEditor.resize()
  }
```

因为`store/index.js`已经导出`Store`实例了，所以直接引入就可以，额外的可以直接挂载到`Class`上

```javascript
this.$store = store
```



#### NextTick的执行顺序

```javascript
const callbacks = []
//.....
//.....
function flushCallbacks () {
  pending = false
  const copies = callbacks.slice(0)
  callbacks.length = 0
  for (let i = 0; i < copies.length; i++) {
    copies[i]()
  }
}
```

一个数组，从0开始执行回调，所以先添加的先执行

如果在组件初始化的时候添加，那么添加顺序为组件不同生命周期的调用顺序

先调用的组件生命周期钩子其`nextTick`回调会放到开头



#### 如何模版直接引用vue实例方法

我的一个`Button`上注册了一个点击事件，我想直接将事件发给父组件，本身不处理

直接fire事件

```vue
<el-button
      id="share"
      icon="el-icon-share"
      size="mini"
      @click="$emit('share')">Share</el-button>
```

发现使用`this.$emit('myevent')`会直接报`this undefined`

其实`Vue`编译成渲染函数之后

```javascript
click: function($event) {
              _vm.$emit("share")
       }
```

`Vue`会将我们的方法会直接绑定到`_vm`上

但是这样只能使用组件方法

```javascript
@click="console.log("Undefined")"
```

上面会直接报错找不到`console`，因为渲染完之后是`_vm.console.log()`，`vm`上没有`console`



**`Vuex `的 `Mutation`最多接收两个参数，如果想要传递多个参数，那么封装成对象传递**



#### `Vue` build之后的`index.html` 静态资源路径问题

如果想要让生成的资源以`/static/js`的`/`开头，那么可以把`webpack.prod.conf.js`的`output.publicPath`的`"/"`改成`""`，`Github`上有说把`config/index.js`中的`assetsPublicPath`从`"/"`改成`""`或者`"./"`，但是我试了却不好使