---
title: '基于vue的埋点方案'
date: 2018-05-07 08:59:01
tags:
---
buried-point基于vue的埋点方案
======
安装Install
------
<pre><code>npm install buried-point</code></pre>

配置configuration
-----------
`main.js`中进行如下配置：
<pre><code>
  import BuriedPoint from 'buried-point'

  let option = {
    router: router,  //vue路由对象
    domain: '127.0.0.1:18080',  //埋点服务器地址
    path: '/nresource/dcsImg'   //埋点请求路径  `/nresource/dcsImg/dcs.log`
  }
  Vue.use(BuriedPoint, option)

</code></pre>

开始使用start
-------------
>1.`v-ready-stat`指令，用于采集页面加载完成时数据
在需要采集数据的vue组件根节点上声明埋点指令`v-ready-stat`

<pre><code>
  &lt;template&gt;
  &lt;div class="button-content" v-ready-stat&gt;
  &lt;/template&gt;
</code></pre>

>如果需要在页面加载时传递数据，`v-ready-stat`指令可以接收`json`格式的参数`{'SI.si_n':'WEB', 'SI.si_x':'002'}`

<pre><code>
  &lt;div class="icon-content" v-ready-stat="{'SI.si_n':'xxx','SI.si_x':'009'}"&gt;
</code></pre>

>2.`v-click-stat`指令，用于采集用户点击事件的数据
在被点击的dom节点上声明埋点指令`v-click-stat`,数据格式`{'SI.si_n':'WEB', 'SI.si_x':'002'}`

<pre><code>
	&lt;el-button type="text" v-data-stat="{'SI.si_n':'WEB', 'SI.si_x':'002'}"&gt;{{message}}&lt;/el-button&gt;
</code></pre>

>3.命令式采集业务码
针对需要在业务逻辑中进行埋点的特殊场景，提供命令式埋点方案；
在vue组件中需要`import`埋点插件`import bp from 'buried-Point'`,在需要埋点的位置调用埋点方法`bp.dcsMultiTrack('SI.si_n', 'WEB_Q', 'SI.si_x', '005')`

<pre><code>
import bp from 'buried-Point'
export default {
  //在vue组件的生命周期钩子mounted阶段，进行埋点
  mounted() {
    bp.dcsMultiTrack('SI.si_n', 'WEB_Q', 'SI.si_x', '005')
  }
}
</code></pre>

