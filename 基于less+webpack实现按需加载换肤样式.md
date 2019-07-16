

### 基于less+webpack实现按需加载换肤样式

翻阅网上常见的换肤方案，大概有以下几种：

1. elementUI基于sass提供全局变量修改实现自定义主题，iview基于less也提供全局变量修改，编译后生成多套css样式文件。

2. antdUI基于less浏览器端提供的modifyVars方法可动态修改全局变量实现实时更新，但需要即时编译消耗开销。

3. 一份css文件使用正则replace所有关键变量后插入style，每次更改时重复全局replace。

   

以上几种多少存在冗余或开销上的问题，本文提供一种基于less与webpack分割异步chunk能力的方案，只针对需要修改的样式进行多分编译，并通过切换body元素上不同的class加载不同样式文件进行覆盖。

| 文件名（假设所有文件在同目录下） | 文件内容                                                     | 备注                                                         |
| -------------------------------- | :----------------------------------------------------------- | ------------------------------------------------------------ |
| common.less                      | .main-color {<br/>  color: @mainColor<br/>}<br/>div {<br/>  .main-color()<br/>} | 维护一个需要换肤的样式文件                                   |
| default.less                     | @mainColor: blue;<br/>@import './common.less';               | 假定该样式为默认主题色                                       |
| default.js                       | *import* './default.less'                                    | 默认主题色入口                                               |
| red.less                         | @mainColor: red;<br/>.red {<br/>  @import './common.less';<br/>} | 假定该样式为切换后的主题色，<br />注意该文件最外层套一层class |
| red.js                           | *import* './red.less'                                        | 切换主题色入口                                               |

注： common.less文件针对每个样式变量提供一个类以便于注入全局使用，如@mainColor变量对应`.main-color`即可在任意组件内自定义样式使用，但要注意该class优先级很低，无法覆盖组件内scoped的class样式。



在入口文件内增加```changeTheme```方法，该方法用于切换加载不同主题色。

```javascript
export default {
  data () {
    return {
      theme: ''
    }
  },
  mounted () {
    this.changeTheme()
  },
  methods: {
    changeTheme () {
      this.theme = this.theme === 'default' ? 'red' : 'default'
      const body = document.body
      body.className = ''
      this.theme !== 'default' && body.classList.add(this.theme)
      import(
        /* webpackChunkName: "theme-[request]" */
        /* webpackInclude: /\.js$/ */
        `./${this.theme}`)
    }
  }
}
```



重点关注```import()```方法，该方法2个重要功能，一个是将指定路径下所有主题色进行一次性根据不同入口独立打包生成独立chunk，这也是为什么每个主题色需要单独提供一个js文件引入，而不是直接import一个less或css文件，因为无法获取到打包后文件hash值。另一个功能是实现按需加载，不会将多个主题色样式打包到同一个包导致体积过大，且后加载的样式会append到最后一个style覆盖先前样式，并且自动缓存。```/* webpackInclude: /\.js$/ */```表示只打包该目录下js结尾的文件。





