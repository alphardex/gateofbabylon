---
title: 将TUIKit从uni-app迁移到taro
author: alphardex
abbrlink: 36637
date: 2024-06-18 17:15:10
tags:
---

[TUIKit](https://cloud.tencent.com/document/product/269/64506)是腾讯的一个基于 Chat SDK 的 UI 组件库，目前支持 H5、小程序、uni-app，但并没有对 Taro 的支持。

由于我的项目是基于 Taro 的，因此我先要把它 uni-app 的版本迁移到 Taro 上去。

<!--more-->

## 准备工作

将[chat-uikit-uniapp](https://github.com/TencentCloud/chat-uikit-uniapp)克隆到本地，把里面原有的 .git 和 sample 删掉，自己初始化一个 git。

当然，在初始化 git 之前，可以把项目名先改成`chat-uikit-taro`，毕竟 Taro 才是主体目标嘛。

```sh
git clone https://github.com/TencentCloud/chat-uikit-uniapp.git
cd chat-uikit-uniapp
rimraf .git
rimraf sample
git init
git add .
git commit -m "init"
```

## 修改库本身

### Taro

把所有`uni.`替换成`Taro.`，注意有些带有`-uni.png`的要手动排除掉。

由于 Taro 需要手动`import`，在所有`vue`文件的`script`部分统一补上。

```vue
<script setup lang="ts">
import Taro from "@tarojs/taro";
...
</script>
```

以及

```vue
<script lang="ts" setup>
import Taro from "@tarojs/taro";
...
</script>
```

### 生命周期

Taro 的生命周期钩子和 uni-app 也不一样，需要找到并替换掉，搜索`@dcloudio/uni-app`，便可找到所有钩子的引入位置，所有钩子替换完后再把`@dcloudio/uni-app`替换成`@tarojs/taro`。注意要把不相关的选项手动排除掉（如`emit`）。

- `onLoad` —— `useLoad`
- `onUnload` —— `useUnload`
- `onReady` —— `useReady`
- `onHide` —— `useDidHide`

### 事件

Taro 的事件处理和 uni-app 也不一样，做以下的替换：

- `$on` —— `eventCenter.on`
- `$off` —— `eventCenter.off`
- `$emit` —— `eventCenter.trigger`

### 编译宏

Taro 的编译宏和 uni-app 并不相同。

先搜索`#ifdef`。

`adapter-vue.ts`中，直接把`vue`指定为`vue3`版本。

```ts
let vueVersion = 3;
let framework = "vue3";
export * from "vue";
console.warn(`[adapter-vue]: vue version is ${vueVersion}`);
export { vueVersion, framework };
```

`entry-chat-only.ts`中，做出以下的修改：

```ts
...
import { TUIChatKit } from "../../index.ts";

export const initChat = (options: Record<string, string>) => {
  ...
  if (process.env.TARO_ENV === "weapp") {
    TUIChatKit.init();
  }
  ...
};
```

再搜索`#ifndef`。

`server.ts`中，将`locale`相关代码注释掉。

```ts
...
// import TUILocales from './locales';
...
// TUITranslateService.provideLanguages({ ...TUILocales });
// TUITranslateService.useI18n();
...
```

### 去除 ts 里对 vue 的引入

Taro 的 ts 文件里不能直接引入 vue 文件，不然会把编译好的字符串全部直出到页面上。

`TUIKit/index.ts`中：

```ts
import { genTestUserSig } from "./debug";
import Server from "./server";
// import TUIComponents, {
//   TUIChat,
//   TUIConversation,
//   TUIContact,
//   TUISearch,
//   TUIGroup,
// } from "./components";
// import TUIKit from "./index.vue";

const TUIChatKit = new Server();
TUIChatKit.init();

export {
  // TUIKit,
  TUIChatKit,
  // TUIComponents,
  // TUIChat,
  // TUIConversation,
  // TUIContact,
  // TUISearch,
  // TUIGroup,
  genTestUserSig,
};
```

### 重命名重名组件

`TUIKit`有一个`Icon.vue`的组件，很容易跟项目所用的组件库里的组件重名，要给它换个名称。

在所有`vue`文件中搜索`Icon.vue`，替换成`IconImage.vue`，同时将`import Icon from`替换成`import IconImage from`，再把`<Icon`替换成`<IconImage`，把`components/common`目录下的`Icon.vue`重命名为`IconImage.vue`。

### 调整样式

搜索`:not(not)`相关样式，把它们全部注释掉。

Taro 不支持局部样式，把所有的`scoped`去除。

在`TUIKit/assets/styles/common.scss`中，把最顶上的样式重置删除（不是注释，直接删）。

```scss
// body, div, ul, ol, dt, dd, li, dl, h1, h2, h3, h4, p {
//   margin:0;
//   padding:0;
//   font-style:normal;

//   /* font:12px/22px"\5B8B\4F53",Arial,Helvetica,sans-serif; */
// }
```

### 解决样式覆盖冲突

上一步把`scoped`去除后，样式就变成了全局样式，可能会产生样式污染的问题，最常见的是`.btn`会污染项目里原本就有的`.btn`。

将`.btn`改个名吧，统一替换成`.tui-btn`，注意要只改样式和`html`，不要影响到其他逻辑里的`btn`。

### 替换导航路径

下面会写到 Taro 的页面跟 uni-app 并不相同，因此导航的路径也会不同。

在所有文件中搜索`/TUIKit/components/`，按以下规则进行替换：

- `/TUIKit/components/TUIChat/video-play` —— `/TUIKit/pages/TUIChat-video-play/TUIChat-video-play`
- `/TUIKit/components/TUIChat/index` —— `/TUIKit/pages/TUIChat/TUIChat`
- `/TUIKit/components/TUIContact/index` —— `/TUIKit/pages/TUIContact/TUIContact`
- `/TUIKit/components/TUIConversation/index` —— `/TUIKit/pages/TUIConversation/TUIConversation`
- `/TUIKit/components/TUIGroup/index` —— `/TUIKit/pages/TUIGroup/TUIGroup`
- `/TUIKit/components/TUISearch/index` —— `/TUIKit/pages/TUISearch/TUISearch`

### 替换变量

将所有`isUniFrameWork`替换成`isTaroFrameWork`。

在`TUIKit/utils/env.ts`中，做以下的修改：

```ts
import Taro from "@tarojs/taro";

// declare const uni: any;

...

export const isTaroFrameWork = typeof Taro !== "undefined";
```

### 解决滚动失效问题

目前 Taro 貌似不支持通过设置`scroll-view`的`scroll-top`来滚动页面，在`TUIKit/components/TUIChat/message-list/index.vue`的末尾加上如下的代码：

```ts
watch(
  () => scrollTop.value,
  (val) => {
    Taro.pageScrollTo({
      scrollTop: val,
      duration: 0,
    });
  }
);
```

## 给项目集成库

主要参考官方文档来一步步做：https://cloud.tencent.com/document/product/269/64506

项目的`package.json`里添加以下依赖：

```json
{
  "dependencies": {
    "@tencentcloud/chat-uikit-engine": "latest",
    "@tencentcloud/tui-core": "latest",
    "@tencentcloud/universal-api": "latest",
    "@tencentcloud/call-uikit-wechat": "latest",
    "@tencentcloud/call-uikit-vue2.6": "latest",
    "@tencentcloud/call-uikit-vue": "latest",
    "@tencentcloud/tui-customer-service-plugin": "latest"
  }
}
```

将`TUIKit`目录整个复制到项目的`src`目录下。

将`node_modules`里的`@tencentcloud/tui-customer-service-plugin`的`tui-customer-service-plugin`目录复制到`TUIKit`目录下。

在`tui-customer-service-plugin/componentsmessage-product-card`中，做出以下修改：

```vue
<script lang="ts">
import Taro from "@tarojs/taro";
...

// eslint-disable-next-line
// declare var uni: any;

...

export default {
  ...
  setup(props: Props) {
    const jumpProductCard = () => {
      if (window) {
        window.open(props.payload.content.url, "_blank");
      } else {
        Taro.navigateTo({
          url: `/TUIKit/pages/TUIChat-web-view/TUIChat-web-view?url=${props.payload.content.url}`,
        });
      }
    };
    ...
  },
};
</script>
```

在`app.ts`中，进行`im`的登录操作。

```ts
const tuiConfig = {
  SDKAppID: 114514,
  userID: "114514",
  userSig: "114514",
};

const App = createApp({
  onShow(options) {},
  // 入口组件不需要实现 render 方法，即使实现了也会被 taro 所覆盖
  onLaunch() {
    TUILogin.login({
      ...tuiConfig,
      useUploadPlugin: true, // If you need to send rich media messages, please set to true.
      framework: `vue3`, // framework used vue2 / vue3
    }).catch(() => {});
  },
});
```

由于 Taro 的单个页面下只支持一个`vue`文件，我们不能直接用`TUIKit/components`里的组件作为页面。

新建一个`TUIKit/pages`目录，再将所有组件对应的目录、以及它下面的`vue`文件和`ts`配置也创建下。

```sh
mkdir src/TUIKit/pages
mkdir src/TUIKit/pages/TUIChat
mkdir src/TUIKit/pages/TUIContact
mkdir src/TUIKit/pages/TUIConversation
mkdir src/TUIKit/pages/TUIGroup
mkdir src/TUIKit/pages/TUISearch
mkdir src/TUIKit/pages/TUIChat-video-play
mkdir src/TUIKit/pages/TUIChat-web-view
touch src/TUIKit/pages/TUIChat/TUIChat.vue
touch src/TUIKit/pages/TUIChat/TUIChat.config.ts
touch src/TUIKit/pages/TUIContact/TUIContact.vue
touch src/TUIKit/pages/TUIContact/TUIContact.config.ts
touch src/TUIKit/pages/TUIConversation/TUIConversation.vue
touch src/TUIKit/pages/TUIConversation/TUIConversation.config.ts
touch src/TUIKit/pages/TUIGroup/TUIGroup.vue
touch src/TUIKit/pages/TUIGroup/TUIGroup.config.ts
touch src/TUIKit/pages/TUISearch/TUISearch.vue
touch src/TUIKit/pages/TUISearch/TUISearch.config.ts
touch src/TUIKit/pages/TUIChat-video-play/TUIChat-video-play.vue
touch src/TUIKit/pages/TUIChat-video-play/TUIChat-video-play.config.ts
touch src/TUIKit/pages/TUIChat-web-view/TUIChat-web-view.vue
touch src/TUIKit/pages/TUIChat-web-view/TUIChat-web-view.config.ts
```

所有的`config.ts`文件里添加统一的配置。

```ts
export default definePageConfig({
  navigationBarTitleText: "消息",
});
```

每个`vue`文件里引入`components`目录下对应的组件（以 TUIChat.vue 为例）

```vue
<script lang="ts" setup>
import TUIChat from "@/TUIKit/components/TUIChat/index.vue";

...
</script>

<template>
  <TUIChat></TUIChat>
</template>
```

最后在`app.config.ts`里添加对应的分包（要看项目，不一定加所有的页面）。

```ts
export default defineAppConfig({
  subPackages: [
    {
      root: "TUIKit",
      pages: [
        "pages/TUIConversation/TUIConversation",
        "pages/TUIChat/TUIChat",
        "pages/TUIChat-video-play/TUIChat-video-play",
        "pages/TUIChat-web-view/TUIChat-web-view",
        "pages/TUIContact/TUIContact",
        "pages/TUIGroup/TUIGroup",
        "pages/TUISearch/TUISearch",
      ],
    },
  ],
  preloadRule: {
    "pages/index/index": {
      network: "all",
      packages: ["TUIKit"],
    },
  },
});
```

## 优化主包体积过大问题

可以用`webpack-bundle-analyzer`插件来直观地查看依赖大小，以进一步地优化体积。

```sh
npm i webpack-bundle-analyzer -D
```

`config/prod.ts`中这么配置：

```ts
module.exports = {
  ...
  mini: {
    webpackChain(chain, webpack) {
      chain
        .plugin("analyzer")
        .use(require("webpack-bundle-analyzer").BundleAnalyzerPlugin, []);
    },
  },
  ...
};
```

这样就能很直观地看到`TUIKit`相关的依赖足足占了 1 个多`M`，WTF！

如果你小程序主包的页面很多且体积很大，就把除了`tab`页以外的页面全放到分包里吧。

项目里有`echarts`这个大头的话，把它也放到分包里吧。

## 其他优化

可以根据项目需求来进一步地魔改`TUIKit/components`里的组件。

## 最后

费了一番功夫，总算迁移完成。

只能说 TX 你做得好啊，你做的好。
