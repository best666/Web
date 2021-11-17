在讲h函数之前，我们先来了解下虚拟dom：

### 虚拟dom

虚拟dom简单来说就是一个普通的 JavaScript 对象，包含tag，props，children三个属性。

```html
<div id="app">
  <p className="text">lxc</p>
</div>
```

上边的HTML代码转为虚拟DOM如下：

```javascript
{
    tag:"div",
    props:{
        id:"app"
    },
    children:[
        {
            tag:"p",
            props:{
                className:"text"
            },
            children:[
                "lxc"
            ]
        }
    ]
}
```

该对象就是所谓的虚拟dom，因为dom对象是属性结构，所以使用JavaScript对象就可以简单表示。而原生dom有许多属性、事件，即使创建一个空div也要付出昂贵的代价。而虚拟dom提升性能的点在于DOM发生变化的时候，通过diff算法对比，计算出需要更改的DOM，只对变化的DOM进行操作，而不是更新整个视图。

### h函数

在 vue 脚手架中，我们经常会看到这样一段代码：

```javascript
  const app = new Vue({
    ··· ···
    render: h => h(App)
  })
```

这个render方法也可以写成这样：

```javascript
  const app = new Vue({
    ··· ···
    render:function(createElement){
        return createElment(App)
    }
  })
```

**h函数就是vue中的createElement方法，这个函数作用就是创建虚拟dom，追踪dom变化的。**

在看一个例子：写一个main.jsx文件 

```javascript
function getVDOM(){
    return (
        <div id="app">
            <p nameProperty="lxc">my name is lxc</p>
        </div>
    )
}
```

使用bable编译之后：

![img](https://img-blog.csdnimg.cn/20190715150357696.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQyNzc4MDAx,size_16,color_FFFFFF,t_70)

**上边代码：最终html代码会被编译成h函数的渲染形式。返回的是一个虚拟DOM对象，通过diff算法，来追踪自己要如何改变真实DOM。**

```javascript
function h(tag,props,...children){//h函数，返回一个虚拟dom对象
    return {
        tag,
        props:props || {},
        children:children.flat()//扁平化数组，降至一维数组
    }
}
```

### createElement函数

我们在看下官方文档所说的createElement函数，它返回的实际上不是一个DOM元素，更准确的名字是：createNodeDescription（直译为——创建节点描述），因为它所包含的信息会告诉vue页面上需要渲染什么样的节点，包括其子节点的描述信息。我们把这样的节点叫做：“虚拟节点（virtual node）”，也常简写为：“VNode”。（看了以上的案例相信在看刚这段话思路会更加清晰）

createElment参数（也就是h函数）：

我们还是以官方文档的解释来讲，createElment函数接受三个参数，分别是：

   参数一：tag（标签名）、组件的选项对象、函数（必选）；

   参数二：一个对象，标签的属性对应的数据（可选）；

   参数三：子级虚拟节点，字符串形式或数组形式，子级虚拟节点也需要使用createElement构建。

来一个小例子：

```vue
<!-- 在模板中渲染 -->
<div id="app">
    {{name}}
    <p>{{age}}</p>
</div>

<!-- 在render函数中渲染 -->
render:function(createElement){
  return createElement("div",{id:"lxc"},[this.name,createElement("p",this.age)])
}
```

上边代码，值得注意的是，如果父元素有文本内容，在render函数中写的时候，内容写在参数三——数组的第一个参数中。（建议大家动手写一写，理解的会更透彻。）

### 官方例子

在来一个官方文档例子：

```vue
<!-- 在模板中渲染 -->
<ul v-if="items.length">
      <li v-for="item in items">{{item.name}}</li>
</ul>
<p v-else>No items</p>

<!-- 在render函数中渲染 -->
data:{
    items:[{name:"l"},{name:"x"},{name:"c"}]
},
render:function(createElement){
  if(this.items.length){
      return createElement("ul",this.items.map(function(ele){
            return createElement("li",ele.name)
      }))
  }else{
      return createElment("p","No items")  
  }
}
```

 