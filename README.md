# vue-tabsbar

#### 介绍
这是一个基于vue、vue-router、elementUI的多选项卡组件2.0版本

#### 软件架构
1. 该组件是通过sessionStorage实现tab状态保持的
2. 该组件是通过监听vue-router的变化以实现tabbar根据路由变化而变化

#### 安装教程

1. 请确保已经引入了vue-router和elementUI
2. 把组件放到你的项目中即可
3. 放入后直接引入即可（vue引入组件的方法这里就不阐述了，直接问度娘即可）


#### 使用说明
调用示例：
```html
<tabs-bar :dont-delete="['index']" :dont-listen="['login']" :multiple="['test']" index-key="name" title-key="meta.title"></tabs-bar>
```
### Props
| 名称    | 类型 | 能否为空 | 默认值 | 说明 |
| ---------- | ---- | ---------- | -------------- |----------------- |
| dont-delete    | Array | √ | [] | 不能删除的tab，路由后，tab就不能被关闭 |
| dont-listen    | Array | √ | [] | 不监听的tab， 组件不会对该路由进行监听，不会根据此路由变化而变化|
| multiple    | Array | √ | [] | 可以重复打开的tab，同一个路由（参数不一致的情况下）可以打开多个tab |
| index-key    | String | × | '' | 该参数是关键，就是路由中定义的唯一的索引值，例如：路由对象的name属性（必须保证这个参数在全部路由对象中是唯一的） |
| title-key    | String | × | '' | 该参数是关键，就是路由中定义的路由名称，例如：在路由对象的meta中定义了title |

### 注意
1. 在开发过程中路由的新增、减少可能会导致tabs队列不正常，只要清除sessionStorage并刷新页面即可，正常上线后不会出现该问题。
2. 目前只在单路由出口的情况下进行测试，未测试过在动态路由、多路由出口、嵌套路由的情况下是否会有BUG。

### 特别说明
1. 本人资历尚浅，而且主要是做后端开发的，不是专业前端。只是刚好有这个需求，所以就自行封装了这个组件，有需要的同学可以拿去用
2. 组件因为只在单路由出口的情况下使用并测试过，其他情况的尚未测试过，有兴趣或者有需要的同学可以自行扩展，代码中已尽可能的写上注释

# License

This content is released under the [MIT](http://opensource.org/licenses/MIT) License.

#### 参与贡献

1.  Fork 本仓库
2.  新建 Feat_xxx 分支
3.  提交代码
4.  新建 Pull Request
