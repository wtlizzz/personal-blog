title: Vue爬坑指南
author: Wtli
tags:
  - VUE
  - echarts
categories:
  - 前端
date: 2020-08-11 08:12:00
---
首先使用VUE-cli新建项目，然后使用ajax请求，调试echarts数据，最后把ajax使用gRPC替换，完成前端。
<!--more-->

### 使用vue-cli新建项目

全局安装vue-cli脚手架

    $ cnpm install vue-cli -g

查看vue-cli是否成功
    
    $ vue list
    
新建vue项目
```
$ vue intit webpack project-name
```


### 入坑指南
解决进入Vue的某些坑～～～
<!--more-->
#### margin-8px
浏览器自动设置body的margin。在写页面的时候，在App.vue文件style中添加
```
body {
  margin: 0;
}

```
#### 地址栏中去掉#
为了让地址栏更加好看，在router下的index.js文件中添加mode。
```
export default new Router({
  mode: 'history',
  routes: [
    {
      path: '/',
      name: 'TabNetMonitor',
      component: TabNetMonitor,
    },
  ],
});
```
#### @意义
在引入文件中，为了能够更加方便的引入，可以使用@标识符，@所代表的目录在webpack.base.conf.js文件中有设置。@路径代表项目中的src文件夹。
```
    alias: {
      'vue$': 'vue/dist/vue.esm.js',
      '@': resolve('src'),
    }
```
#### require报错
使用eslint来规范自己代码，发现在data中使用require报错，写一个图片，需要引入本地的url，网上都是直接使用require引入。
```
src: require('../../assets/logo2.jpeg'),
```
现需要改成使用import引入。
```
import logo2 from '../../assets/logo2.jpeg';
export default {
  data() {
    return {
      src: logo2,
      fit: 'contain',
    };
  },
};
```
#### 使用webstorm配置eslint
还是熟悉的webstorm好用，更熟悉一些，但是刚从vscode转过来，还需要重新配置eslint。  
- 首先是在&lt;script&gt;和&lt;style&gt;标签下不添加两个空格，如下图，添加两个标签。
![avrTij.png](https://s1.ax1x.com/2020/08/12/avrTij.png)
- 在Tabs and Indents中将4个空格改成2个

<font color=gray>小知识：使用
```
&lt;script&gt;
```
转义字符显示
```
<script>
```
</font>
  
#### 相对布局
- flex-direction不起作用
```
flex-direction: row;
justify-content: center;
```
需要在父容器中添加
```
display: flex;
```
- align-content不起作用
```
align-content: center;
```
align-content是相对于多行情况下，在单行情况下使用：
```
align-items: center;
```

<font color=gray>淦！～～和之前ReactNative咋不一样呢</font>
#### 固定布局居中
使用fix固定布局，配置居中将margin配置成auto，将top/left/buttom/right设置为0。
```
.main {
  position: fixed;
  margin: auto;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  width: 800px;
  height: 800px;
}
```


### 配置scss 
实在是想写css的嵌套，看了半天还是scss好用，现在Vue中集成scss，安装必要组件
```
$ npm install node-sass --save
$ npm install sass-loader --save
$ npm install style-loader --save
$ npm install sass-resources-loader --save
```
在build文件夹下的webpack.base.conf.js的rules里面添加配置
```
{
 　test: /\.sass$/,
　 loaders: ['style', 'css', 'sass']
}
```
如下图所示：
![avAEh6.png](https://s1.ax1x.com/2020/08/12/avAEh6.png)

<font color=gray>版本过高还会报错如下图，解决办法,安装低版本。
```
$ npm install sass sass-loader@6.0.6
```
  。。。淦</font>
![avV279.png](https://s1.ax1x.com/2020/08/12/avV279.png)
#### scss使用
- 使用$声明变量。

```
$link-color: blue;
a {
  color: $link_color;
}
//编译后
a {
  color: blue;
}
```


### 使用Echarts绘制简单Demo
- 先创建一个区域，设置高度，宽度，width: 600px;height:400px;
```
<template>
<div id="main" style="width: 600px;height:400px;">
</div>
</template>
```
- 第一种写法，使用window.onload()：  
window.onload() 方法用于在网页加载完毕后立刻执行的操作，即当 HTML 文档加载完毕后，立刻执行某个方法。  
window.onload() 通常用于 &lt;body> 元素，在页面完全载入后(包括图片、css文件等等)执行脚本代码。
```
<script>
window.onload = function () {
  const myChart = echarts.init(document.getElementById('main'));
  const option = {
    title: {
      text: 'ECharts 入门示例',
    },
    tooltip: {},
    legend: {
      data: ['销量'],
    },
    xAxis: {
      data: ['衬衫', '羊毛衫', '雪纺衫', '裤子', '高跟鞋', '袜子'],
    },
    yAxis: {},
    series: [{
      name: '销量',
      type: 'bar',
      data: [5, 20, 36, 10, 10, 20],
    }],
  };
  myChart.setOption(option);
};
</script>
```
- 第二种写法，更符合Vue使用，在Vue生命周期函数，mounted方法中使用。
```
<script>
mounted() {
  this.displayChart();
},
methods: {
  displayChart() {
    const myChart = echarts.init(document.getElementById('main'));
    const option = {
      title: {
        text: 'ECharts 入门示例',
      },
      tooltip: {},
      legend: {
        data: ['销量'],
      },
      xAxis: {
        data: ['衬衫', '羊毛衫', '雪纺衫', '裤子', '高跟鞋', '袜子'],
      },
      yAxis: {},
      series: [{
        name: '销量',
        type: 'bar',
        data: [5, 20, 36, 10, 10, 20],
      }],
    };
    myChart.setOption(option);
  },
},
</script>
```
**顺带记录一下Vue生命周期**  
差不多一共8个生命周期钩子函数
- beforeCreate：不能在这个方法里添加请求，加载不到method里面的方法。～
- create：在这个方法里添加axios请求，返回的数据竟然比mounted里面慢，怀疑是axios异步的问题。
- beforeMount
- mounted
- beforeUpdate
- updated
- beforeDestroy
- destroyed
[![az9yge.png](https://s1.ax1x.com/2020/08/13/az9yge.png)](https://imgchr.com/i/az9yge)

### Vue使用Ajax发送请求
- 安装axios
```
$ npm install axios -S
```
- 安装qs，方便将json转换成data
```
$ npm install qs.js --save
```
- 全局注册,在main.js中增加配置
```
Vue.prototype.$axios = axios;
Vue.prototype.qs = qs;
```
- 使用安全的方法读取数据
```
const { data: items2, data2 = 0 } = this.netInfo;
```
### SpringBoot配置跨域
简单解决跨域问题，日后再详细研究。新建配置类，重写addCorsMappings方法。
```
@Configuration
public class CORSConfiguration extends WebMvcConfigurationSupport {
    @Override
    protected void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowedMethods("GET", "HEAD", "POST","PUT", "DELETE", "OPTIONS")
                .allowedHeaders("*")
                .exposedHeaders("access-control-allow-headers",
                        "access-control-allow-methods",
                        "access-control-allow-origin",
                        "access-control-max-age",
                        "X-Frame-Options")
                .allowCredentials(false).maxAge(3600);
        super.addCorsMappings(registry);
    }
}
```
### 使用Echarts绘制圆形拓扑关系图
1. 使用Math函数计算Y坐标。  
> Math.pow() 函数返回基数（base）的指数（exponent）次幂，即 base<sup>exponent</sup>。  
> Math.sqrt() 函数返回一个数的平方根。
```
const getY = function (x) {
    const y = Math.sqrt((1 - Math.pow(x / 38, 2)) * Math.pow(30, 2));
    return y;
};
const getOutY = function (x) {
    const y = Math.sqrt((1 - Math.pow(x / 50, 2)) * Math.pow(42, 2));
    return y;
};
```
当x=0时，y=1；显示在圆心的正上方。圆心的坐标是（0，0）。
2.

### Vue部署
如果route使用的history模式，需要去掉，history模式需要配置服务器。
- 部署到本地Tomcat上
```
$ npm run build
```
> 将生成的dist文件夹，拷贝到tomcat/webapp中，启动tomcat，启动命令在tomcat/bin文件夹中运行.sh文件
```
$ ./startup.sh   // 启动
$ ./shutdown.sh   //停止
```
- 部署到github Pages/gitee Pages中  
<font color=gray>由于github连接速度太慢，将github替换为gitee，gitee不好的地方就是不能绑定域名，上传文件不能自动部署，gitee对hexo的编译太慢。。其他的都还好。</font>  
> 修改config文件夹中的index.js文件,将assetsPublicPath: '/'修改为'./'，更改资源访问的目录。
```
build: {
    assetsPublicPath: './',
}
```