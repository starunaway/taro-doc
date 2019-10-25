# Readme

[Taro 官方文档](https://nervjs.github.io/taro/docs/taroize.html)  
[微信小程序 官方文档](https://developers.weixin.qq.com/miniprogram/dev/framework/)

## Taro CLI

### init

```bash
# 使用 npm 安装 CLI
$ npm install -g @tarojs/cli

# 使用 yarn 安装 CLI
$ yarn global add @tarojs/cli

# 使用命令创建模板项目
$ taro init myApp
```

### taro 版本更新

```bash
# taro
$ taro update self
# npm
npm i -g @tarojs/cli@latest
# yarn
yarn global add @tarojs/cli@latest
```

### 更新项目中 Taro 相关的依赖

```bash
$ taro update project
```

### Issue

1. 回到某个版本后

```bash
  # 使用 npm 安装 CLI
  $ npm install -g @tarojs/cli@1.3.9
  # OR 使用 yarn 安装 CLI
  $ yarn global add @tarojs/cli@1.3.9
  # OR 安装了 cnpm，使用 cnpm 安装 CLI
  $ cnpm install -g @tarojs/cli@1.3.9
```

使用 `taro update project` 并不能回退项目里面的 taro 版本

## 项目目录

```md
├── config 配置目录
| ├── dev.js 开发时配置
| ├── index.js 默认配置
| └── prod.js 打包时配置
├── dist 微信开发者工具打开的目录
├── src 源码目录
| ├── actions
| ├── asserts 静态资源
| ├── components 公共组件目录
| ├── constants url
| ├── pages 页面文件目录
| | ├── index index 页面目录
| | | ├── index.js index 页面逻辑
| | | └── index.css index 页面样式
| | ├── ···
| | └── ···
| ├── utils 公共方法库
| | ├── fetch request 请求
| | ├── reduxUtils 重新封装了接口返回字段
| | ├── share 小程序分享，可以扩展统计用户拉新能力
| | └── log 从基础库 2.8.1 开始支持
| ├── app.css 项目总通用样式
| └── app.js 项目入口文件，appId
└── package.json
```

## 生命周期

| taro                 | wx       | 作用                                              |
| -------------------- | -------- | ------------------------------------------------- |
| componentWillMount   | onLaunch | 程序被载入                                        |
| componentDidMount    | onLaunch | 程序被载入，在 componentWillMount 后执行          |
| componentDidShow     | onShow   | 程序启动，或从后台进入前台显示时触发              |
| componentDidHide     | onHide   | 程序从前台进入后台时触发 (被动触发)               |
| componentWillUnmount | onUnload | 页面卸载时触发，redirectTo 或 navigateBack,(主动) |

```js
import Taro, {Component} from '@tarojs/taro';
import {View, Text} from '@tarojs/components';
import './index.scss';

export default class Index extends Component {
  config = {
    navigationBarTitleText: '首页'
  };

  componentWillMount() {}

  componentDidMount() {}

  componentWillUnmount() {}

  componentDidShow() {}

  componentDidHide() {}

  render() {
    return (
      <View className='index'>
        <Text>1</Text>
      </View>
    );
  }
}
```

## 配置

总的配置在[app.js](https://developers.weixin.qq.com/miniprogram/dev/reference/configuration/app.html)文件内

### 常用的配置

| 属性            | 描述               | 补充说明                                     |
| --------------- | ------------------ | -------------------------------------------- |
| pages           | 页面路径列表       | 相当于 routes                                |
| tabBar          | 底部 tab 栏的表现  | icon 尺寸和预期不符，现在是 60\*60，可自定义 |
| usingComponents | 全局自定义组件配置 | 和 page 一样，需要先在这里注册               |

### 页面配置

[页面配置](https://developers.weixin.qq.com/miniprogram/dev/reference/configuration/page.html)定义每个页面，每个页面都可以定义不同的样式和行为

**webview 组件中，不支持自定义导航栏，会出现默认导航栏和自定义导航栏共存的情况**

## 组件

1. 组件样式隔离，不受附件格式的影响

1. taro 不支持多层 if 生成元素

   ```js
   // 编译出错
   if (condition1) {
     if (something) {
       return something1;
     } else {
       return something2;
     }
   }
   // 应当这么写
   if (condition1) {
     let el;
     if (something) {
       el = something1;
     } else {
       el = something2;
     }
     return el;
   }
   ```

1. 使用小程序原生第三方组件

   > 如果使用了微信小程序的第三方组件，那么项目只能转换成微信小程序

   > 需要单独在页面的 config 中指定

   ```js
   import Taro, {Component} from '@tarojs/taro';
   import {View} from '@tarojs/components';
   function initChart() {
     // ....
   }
   export default class Menu extends Component {
     static defaultProps = {
       data: []
     };
     config = {
       // 定义需要引入的第三方组件
       usingComponents: {
         'ec-canvas': '../../components/ec-canvas/ec-canvas' // 书写第三方组件的相对路径
       }
     };
     constructor(props) {
       super(props);
       this.state = {
         ec: {
           onInit: initChart
         }
       };
     }
     componentWillMount() {
       console.log(this); // this -> 组件 Menu 的实例
     }
     render() {
       return (
         <View>
           <ec-canvas id='mychart-dom-area' canvas-id='mychart-area' ec={this.state.ec}></ec-canvas>
         </View>
       );
     }
   }
   ```

## 原生代码混写

> 需要将原生的页面、组件代码放入 src 目录下，随后在 入口文件 app.js 中定义好 pages 配置指向对应的原生的页面

实际上还需要在 app.js 中添加 build 参数的命令才能生效(现有的两个页面是直接复制一份代码到目录中)。

## Issues

1. 微信不支持修改光标颜色，在 iphone Android 和微信开发者工具中表现不一致

1. taro build 的时候偶尔会失败，重新 build -> 成功

1. 微信开发者工具 使用 webview 打开业务域名，会提示找不到 appId，但在手机上是好的

1. wxss 使用的尺寸单位是 rpx，将屏幕宽度恒定划为 750rpx; 使用 rpx 进行适配

1. 业务域名和服务域名不一样，都只能配置 https 或 wss。使用 webview 打开页面及 webview 页面调用的服务都需要配置到业务域名中，需要在域名根目录下添加微信的校验文件
