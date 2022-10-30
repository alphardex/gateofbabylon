title: 蒜物语——HTML与WebGL的融合之旅
author: alphardex
abbrlink: 8368
date: 2022-10-29 15:52:04
tags:
---
哟哈喽！这里是alphardex。

[Lycoris Recoil](https://www.agemys.net/detail/20220142)，又称“莉可丽丝”或“石蒜”，是笔者最近追完的一部番。这部番主要还是看美少女贴贴的日常，不论是OP的击股之交，泷奈的Sakana~还是千束神一般的闪避能力等都令我记忆犹新。尽管后面几集剧情可能有点争议，但不影响我对这部番的喜爱。

目前还有在追的一部新番[孤独摇滚](https://www.agemys.net/detail/20220121)也不错，女主社恐的性格真的跟我十分相似（悲）。

最近我心血来潮，想为石蒜这部动画做一个自制的人物介绍页面，素材借用的是[动画官网的素材](https://lycoris-recoil.com/character/)，原本想做一个简单的包含CSS动画的Swiper滑动展示页，但转念一想，既然一直在研究WebGL，为何不搞点更炫的东西呢？于是乎，借助我的框架[kokomi.js](https://github.com/alphardex/kokomi.js)之力，我成功地将一个普通的HTML页面一步步地转化为了一个拥有酷炫交互的WebGL页面。

页面链接在下方，点击右上的logo全屏观看，效果最佳：
https://code.juejin.cn/pen/7159562292110032900

本文并不会详细地去教大家如何完整地做出整个页面的效果，而是来谈谈HTML转WebGL的方法以及WebGL特效的一些常用技法。

<!--more-->

## HTML

HTML是网页最基本的框架，相信大部分时候我们前端都是在跟她打交道吧~

首先，我像往常一样，勾勒了页面最基本的形态，将静态页面排好

由于是角色展示页，有多个角色的信息需要展示在一个页上，我使用了[Swiper](https://swiperjs.com/)作为页面滑动的核心，再加上一些CSS动画，一个基本的原型就完成了

细心的读者会发现，如果将`--webgl--`上方的2行代码解除注释，就会看到最基本的Swiper滑动页效果

![swiper.gif](https://s2.loli.net/2022/10/29/uS6XdB3iUapZcPQ.gif)

这部分比较简单，因此一笔带过。接下来，让我们开始向重点——WebGL迈进

## 图片全屏动画特效

在页面上有这么一个交互：点击一张缩略图时，它会自动放大至占满全屏，并且中间的动画效果比较3D化（类似MAC的那种效果）

借助kokomi.js的[Gallery组件](https://github.com/alphardex/kokomi.js/blob/main/src/web/gallery.ts)，我们就能很方便地将所有的`img`图片元素转化到WebGL的世界中，并用顶点着色器和片元着色器来实现特效

```js
const gallary = new kokomi.Gallery(base, {
  vertexShader, // 顶点着色器
  fragmentShader, // 片元着色器
  materialParams: {
    transparent: true, // 开启透明通道
  },
  scroller, // 设定页面滚动控制器
  elList: [...document.querySelectorAll("img:not(.webgl-fixed)")], // 需要同步的HTML元素
  uniforms: {
    // uniform变量
    ...
  },
});
```

首先，我们先把2个数据作为uniform传递给shader：图片在DOM世界的大小`uMeshSize`和图片位置`uMeshPosition`

```js
this.gallary.makuGroup.makus.forEach((maku) => {
  maku.mesh.material.uniforms.uMeshSize.value = new THREE.Vector2(
    maku.el.clientWidth,
    maku.el.clientHeight
  );
  maku.mesh.material.uniforms.uMeshPosition.value = new THREE.Vector2(
    maku.mesh.position.x,
    maku.mesh.position.y
  );
});
```

并且声明一个`uProgress`的变量，表示该动画的进度

接下来主要编写顶点着色器`vertexShader`，实现全屏动画`fullscreen`函数

```glsl
vec3 fullscreen(vec3 p){
    // get progress
    float pr=uProgress;
    
    // scale to view size
    vec2 scale=mix(vec2(1.),iResolution/uMeshSize,pr);
    p.xy*=scale;
    
    // move to center
    p.x+=-uMeshPosition.x*pr;
    p.y+=-uMeshPosition.y*pr;
    
    // z
    p.z+=pr;
    
    return p;
}
```

从小图缩放到大图，最主要的还是一个缩放比例的插值过程，从原本的1变化到全屏与图片本身的比例，即`iResolution/uMeshSize`，再用动画进度`pr`进行插值即可

缩放至全屏后，图片依旧是停留在原地的，我们需要将它挪到画面的中心，给图片的xy坐标分别减去图片位置`uMeshPosition`的xy与动画进度`pr`的积

js中利用gsap控制动画进度即可

![1-1.gif](https://s2.loli.net/2022/10/30/OGF7V6nhkyNY5UJ.gif)

nice，已经成功地实现了全屏动画效果，但这样还是有点普通，可以再稍微灵动一下

目前的动画进度`pr`是比较同步的，我们需要让它更加交错一点，可以尝试用图片uv坐标的x轴来交错它

```glsl
float getProgress2(float pr,vec2 uv){
    float activation=uv.x;
    float latestStart=.5;
    float startAt=activation*latestStart;
    pr=smoothstep(startAt,1.,pr);
    return pr;
}
```

再者，我们也可以对图片坐标本身进行一些变换，比如翻转，尝试将坐标的x轴翻转下试试，同时，也别忘了翻转下uv坐标的x，不然图片会倒过来显示哦

```glsl
vec3 flipX(vec3 p,float pr){
    p.x=mix(p.x,-p.x,pr);
    return p;
}

vec2 flipUvX(vec2 uv,float pr){
    uv.x=mix(uv.x,1.-uv.x,pr);
    return uv;
}

vec3 fullscreen(vec3 p){
    // copy uv
    vec2 newUv=uv;
    
    // get progress
    float pr=getProgress2(uProgress,uv);
    
    // scale to view size
    vec2 scale=mix(vec2(1.),iResolution/uMeshSize,pr);
    p.xy*=scale;
    
    // other transforms
    p=flipX(p,pr);
    
    float latestStart=.5;
    float stepVal=latestStart-pow(latestStart,3.);
    newUv=flipUvX(newUv,step(stepVal,pr));
    
    // get uv
    vUv=newUv;
    
    // move to center
    p.x+=-uMeshPosition.x*pr;
    p.y+=-uMeshPosition.y*pr;
    
    // z
    p.z+=pr;
    
    return p;
}
```

![1-2.gif](https://s2.loli.net/2022/10/30/TDNscJg9oSvFVWh.gif)

如此，我们的全屏动画效果就完美地实现了，当然，除了翻转效果外还会有很多其他的派生变体，这就等读者去自己发掘啦~

## 文字网格式显现特效

页面刚显现时我们可以看到，文字会以网格带阴影的形式显现出来，有点赛博朋克的风格

网页一般是由图片和文字组成的，既然图片能同步到WebGL世界，那么文字其实也可以，这里就要用到kokomi.js的[MojiGroup组件](https://github.com/alphardex/kokomi.js/blob/main/src/web/moji.ts)，用法和Gallery组件大同小异，只不过是同步对象变成了文字而已

这里为什么不用中文字体？因为暂时找不到中文字体的cdn T_T

```js
const mg = new kokomi.MojiGroup(base, {
  vertexShader: vertexShader,
  fragmentShader: fragmentShader,
  scroller,
  uniforms: {
    ...
  },
});
```

将句子长度`uGridSize`传入shader

```js
this.mg.mojis.forEach((moji) => {
  moji.textMesh.mesh.material.uniforms.uGridSize.value =
    moji.textMesh.mesh._private_text.length;
});
```

利用[噪声函数](https://gist.github.com/patriciogonzalezvivo/670c22f3966e662d2f83#generic-123-noise)配合`floor`函数来形成网格状图案

```glsl
vec2 grid=uGrid;
grid.x*=uGridSize;
vec2 gridP=vec2(floor(grid.x*p.x),floor(grid.y*p.y));
float pattern=noise(gridP);
```

定义动画进度`uProgress`，并用网格状图案对文字颜色`uTextColor`进行插值处理

```glsl
float map(float value,float min1,float max1,float min2,float max2){
    return min2+(value-min1)*(max2-min2)/(max1-min1);
}

float saturate(float a){
    return clamp(a,0.,1.);
}

float getMixer(vec2 p,float pr,float pattern){
    float width=.5;
    pr=map(pr,0.,1.,-width,1.);
    pr=smoothstep(pr,pr+width,p.x);
    float mixer=1.-saturate(pr*2.-pattern);
    return mixer;
}

void main(){
    ...
    vec4 col=vec4(0.);
    
    vec4 l0=vec4(uShadowColor,1.);
    float pr0=uProgress;
    float m0=getMixer(p,pr0,pattern);
    col=mix(col,l0,m0);
    
    ...
}
```

同样的可以定义文字阴影`uShadowColor`，以同样的方式定义另一个动画进度`uProgress1`并进行插值处理

```glsl
vec4 l1=vec4(uTextColor,1.);
float pr1=uProgress1;
float m1=getMixer(p,pr1,pattern);
col=mix(col,l1,m1);
```

js中利用gsap控制2个动画进度即可，可以用`stagger`属性来错开文字阴影的进度，以显得更加生动

最后的效果是这样的：

![2.gif](https://s2.loli.net/2022/10/30/18EGivoy6wTulLz.gif)

## 背景微粒特效

我们通过观察，可以注意到画面的背景是朦胧的微粒漂浮效果，能给页面整体进行一种良好的点缀

可以通过kokomi.js的[CustomPoints组件](https://github.com/alphardex/kokomi.js/blob/main/src/shapes/customPoints.ts)来实现微粒特效

首先要定义好微粒的`geometry`，这里传入了随机的顶点位置数据

```js
const geometry = new THREE.BufferGeometry();

const posBuffer = kokomi.makeBuffer(count, () =>
  THREE.MathUtils.randFloatSpread(3)
);
kokomi.iterateBuffer(posBuffer, posBuffer.length, (arr, axis) => {
  arr[axis.x] = THREE.MathUtils.randFloatSpread(3);
  arr[axis.y] = THREE.MathUtils.randFloatSpread(3);
  arr[axis.z] = 0;
});

geometry.setAttribute("position", new THREE.BufferAttribute(posBuffer, 3));
```

初始化微粒对象

```js
const cm = new kokomi.CustomPoints(base, {
  baseMaterial: new THREE.ShaderMaterial(),
  geometry,
  vertexShader: vertexShader,
  fragmentShader: fragmentShader,
  materialParams: {
    side: THREE.DoubleSide,
    transparent: true,
    depthWrite: false,
  },
  uniforms: {
    ...
  },
});
```

在顶点着色器中，我们可以通过[噪声函数](https://github.com/hughsk/glsl-noise)来扭曲微粒的顶点坐标，以实现微粒随机飘动的效果

```glsl
vec3 distort(vec3 p){
    float speed=.1;
    float noise=cnoise(p)*.5;
    p.x+=cos(iTime*speed+p.x*noise*100.)*.2;
    p.y+=sin(iTime*speed+p.x*noise*100.)*.2;
    p.z+=cos(iTime*speed+p.x*noise*100.)*.5;
    return p;
}

void main(){
    vec3 p=position;
    
    vec3 dp=distort(p);
    
    csm_Position=dp;
    
    vUv=uv;
    
    ...
}
```

在片元着色器中，我们可以定义好微粒的形状和颜色，这里形状用了圆形，颜色是通过uniform传入的

```glsl
float circle(float d,float size,float blur){
    float c=smoothstep(size,size*(1.-blur),d);
    float ring=smoothstep(size*.8,size,d);
    c*=mix(.7,1.,ring);
    return c;
}

void main(){
    float distanceToCenter=distance(gl_PointCoord,vec2(.5));
    float strength=circle(distanceToCenter,.5,.4);
    
    vec3 col=uColor;
    
    csm_DiffuseColor=vec4(col,strength);
}
```

由于微粒效果本身的大小并不跟外面的`ScreenCamera`一致，因此创建了一个[ScreenQuad组件](https://github.com/alphardex/kokomi.js/blob/main/src/shapes/screenQuad.ts)，再通过[RenderTexture组件](https://github.com/alphardex/kokomi.js/blob/main/src/renderTargets/renderTexture.ts)将其渲染到了整个屏幕上

最后实现效果如下，特地把微粒个数给调多了点，以让它更加明显：

![3.gif](https://s2.loli.net/2022/10/30/4DV7NBbGhAsRjZ3.gif)

## 画面凸起特效

当我们点击右上角的头像切换人物或者用鼠标来滚动画面时，我们能看到那画面凸起的转场特效，可以说是很天马行空了

这种凸起特效是用后期处理实现的。kokomi.js的[CustomEffect组件](https://github.com/alphardex/kokomi.js/blob/main/src/postprocessing/customEffect.ts)能轻松实现后期处理的效果，同样只需传入2个着色器以及uniform变量，不过以下的2个特效都只跟片元着色器有关

以圆的方式来扭曲整个屏幕的顶点，这就是凸起效果的要义

居中uv坐标，获取中心点向量，并用它对uv坐标进行偏移操作，如果直接乘上`center`是内凹，外凸的话反转一下就行了

```glsl
vec2 centerUv(vec2 uv){
    uv=uv*2.-1.;
    return uv;
}

vec2 distort(vec2 p){
    vec2 cp=centerUv(p);
    float center=distance(p,vec2(.5));
    vec2 offset=cp*(1.-center)*uProgress;
    p-=offset;
    return p;
}

void main(){
    vec2 p=vUv;
    p=distort(p);
 
    vec4 tex=texture(tDiffuse,p);
    
    vec4 col=tex;
    
    gl_FragColor=col;
}
```

![4.gif](https://s2.loli.net/2022/10/30/KZd3Jb6N9hFH4GY.gif)

## 画面RGB扭曲特效

当我们用鼠标随意在画面上移动时，可以观察到一种微妙的颜色变换效果，这就是著名的RGB扭曲特效

这里也用了全屏的后期处理，实现方法其实很简单：对画面进行3次采样，采样前偏移3个uv坐标，再分别获取采样后的rgb三个通道，将其合并即可

```glsl
vec4 RGBShift(sampler2D t,vec2 rUv,vec2 gUv,vec2 bUv){
    vec4 color1=texture(t,rUv);
    vec4 color2=texture(t,gUv);
    vec4 color3=texture(t,bUv);
    vec4 color=vec4(color1.r,color2.g,color3.b,color2.a);
    return color;
}

void main(){
    vec2 p=vUv;
    p=distort(p);
    
    float mask=1.-getCircle(uMaskRadius/uDevicePixelRatio);
    float r=mask*uMouseSpeed*.5;
    float g=mask*uMouseSpeed*.525;
    float b=mask*uMouseSpeed*.55;
    vec4 tex=texture(tDiffuse,p);
    
    vec4 col=tex;
    
    gl_FragColor=col;
}
```

这里做成了鼠标悬浮到某块区域才触发的特效，因此有了以下的`getCircle`函数，比较通用，在我之前写过的一些特效中也有用到

```glsl
float circle(vec2 st,float r,vec2 v){
    float d=length(st-v);
    float c=smoothstep(r-.2,r+.2,d);
    return c;
}

float getCircle(float radius){
    vec2 viewportP=gl_FragCoord.xy/iResolution/uDevicePixelRatio;
    float aspect=iResolution.x/iResolution.y;
    
    vec2 m=iMouse.xy/iResolution.xy;
    
    vec2 maskP=viewportP-m;
    maskP/=vec2(1.,aspect);
    maskP+=m;
    
    float r=radius/iResolution.x;
    float c=circle(maskP,r,m);
    
    return c;
}
```

![5.gif](https://s2.loli.net/2022/10/30/y4nhKr69PwGZlIX.gif)

## 最后

希望本文能给你创作新特效的灵感，keep creating~