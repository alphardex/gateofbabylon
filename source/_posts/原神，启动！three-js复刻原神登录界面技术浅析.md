title: 原神，启动！three.js 复刻原神登录界面技术浅析
author: alphardex
abbrlink: 10700
date: 2023-10-16 08:45:10
tags:
---
之前看到有人在浏览器端复刻了原神的登录界面，效果非常还原。

本文就让我们一起来看看这种效果是如何实现的，主要分析`Shader`相关的部分。

怀着学习的目的，我自己也写了一版，观看地址：https://genshin-replica.miaohezi.com/

![b86976a6-1228-4658-bae1-5b842e588206](https://s2.loli.net/2023/10/16/IK3H2qUd4o7BiEn.png)

<!--more-->

## 背景

背景是一个渐变色，由 3 种颜色组成，可以用`Shader`的`smoothstep`函数配合`UV`坐标的`y`分量画出来。

```glsl
float y=1.-uv.y;
float mask1=1.-smoothstep(0.,stop1,y);
float mask2=smoothstep(0.,stop1,y)*(1.-smoothstep(stop1,stop2,y));
float mask3=smoothstep(stop1,stop2,y);
col+=col1*mask1;
col+=col2*mask2;
col+=col3*mask3;
```

![f1226134-662b-40e9-a94a-83dd71c68baf](https://s2.loli.net/2023/10/16/ziodAgqyUZ1658Q.png)

整个`Shader`代码我放到`Shadertoy`上了，预览地址：https://www.shadertoy.com/view/dstBzj

## 背景云

背景云由两种平面网格组成，它们有属于自己的遮罩纹理。

![20231016100627](https://s2.loli.net/2023/10/16/Ax4TmUJwBkW72Vi.png)

在`Shader`中，采样遮罩纹理，获取里面的透明度数据，将其作为`alpha`通道的值，再定义一个主颜色即可。

```glsl
vec2 uv=vUv;
vec4 tex=texture(uTexture,uv);
float alpha=tex.r*.4;
vec3 col=vec3(1.8);
gl_FragColor=vec4(col,alpha);
```

![68b8b5c3-8c85-4af4-9499-73a845391ac7](https://s2.loli.net/2023/10/16/wNh46VImR2rgoFK.png)

这是其中一种平面的写法，它只有一种颜色。

另一个平面有两种颜色，是因为用了`mix`函数将 2 种颜色混合到了一起。

```glsl
vec3 col=vec3(.090,.569,.980);
col=mix(col,vec3(.93),vec3(pow(tex.r,.4)));
```

![d18fc630-40a9-4f42-803a-ad31d8350a53](https://s2.loli.net/2023/10/16/A35O7PXSLoVawlQ.png)

## 柱子

柱子的原始信息是一大堆杂乱的网格数据，每条数据包含了模型名、位置、旋转、缩放数据，并且值都是随机的。

![20231016095144](https://s2.loli.net/2023/10/16/ZNAHeyc9LrXRtqz.png)

我们需要将这些信息分个组，大概分成这样：

![20231016095257](https://s2.loli.net/2023/10/16/IzM51WDyEn9QTdf.png)

每个组包含了模型名，实例网格列表和模型的组成网格列表。

在`three.js`中，当我们需要同时创建大量的网格，并且每个网格的位置数据不同时，需要用到实例化网格[THREE.InstancedMesh](https://threejs.org/docs/#api/zh/objects/InstancedMesh)。

我们根据这些组，创建多个实例化网格，并且在渲染中不断地同步它们的位置数据。

![e0200ce4-0b72-4938-a0ab-2dc767887bd9](https://s2.loli.net/2023/10/16/lpEe6Y9MSQ3RAKI.png)

## 云层

由于云层也有很多随机的位置数据，因此也要用到实例化网格。

材质方面，也用到了遮罩纹理。

![20231016102305](https://s2.loli.net/2023/10/16/qDJXBOLRKrjSdET.png)

在采样纹理前，我们要将`UV`坐标在顶点着色器中随机化，这样云层就能呈现出不同的图案。

```glsl
vec2 distortUV(vec2 uv,vec2 offset){
    vec2 wh=vec2(2.,4.);
    uv/=wh;
    float rn=ceil(random(offset)*wh.x*wh.y);
    vec2 cell=vec2(1.,1.)/wh;
    uv+=vec2(cell.x*mod(rn,wh.x),cell.y*(ceil(rn/wh.x)-1.));
    return uv;
}

void main(){
    ...
    vUv=distortUV(uv,instPosition.xy);
    ...
}
```

> 注：这里为了凸显出云层本身，临时把柱子和背景云隐藏掉了。

![40518cb7-5d59-444d-9bd6-921f13c0ad6c](https://s2.loli.net/2023/10/16/VLJhsPTE6RyfH4X.png)

## 光束

材质也用到了遮罩纹理。

![af56bd20-1873-4113-9e93-d5b7b3e59a4e](https://s2.loli.net/2023/10/16/PGvVOfL3mICQbYR.png)

直接采样的话会感觉光的边缘太直了，用`smoothstep`生成遮罩来虚化它吧。

```glsl
float mask1=1.;
mask1*=smoothstep(0.,.5,uv.y);
mask1*=smoothstep(0.,.1,uv.x);
mask1*=smoothstep(1.,.9,uv.x);
```

![shadertoy](https://s2.loli.net/2023/10/16/smBMqiG3SC8OjWt.png)

应用遮罩后就变成了这样：

![94ef1a54-0e4a-4cc5-b8aa-763cd5902e32](https://s2.loli.net/2023/10/16/4fXnYProlOWiLMg.png)

## 星星

用到了`three.js`的粒子系统`THREE.Points`。每一个粒子的位置数据都是随机生成的。

![20231016104602](https://s2.loli.net/2023/10/16/bdQsDpv5jFAVRJK.png)

它的遮罩纹理包含了 3 种图案，在`Shader`中也要随机化`UV`，这样星星就能显示出不同的形状。

```glsl
float seed=random(vRandom);
float randId=floor(seed*3.)/3.;
float mask=texture(uTexture,vec2(uv.x/3.+randId,uv.y)).r;
```

![4b0a94ed-6177-4a78-ac50-85dc048ef01e](https://s2.loli.net/2023/10/16/2ucetRM7jJiD5Cr.png)

## 雾气

在`Shader`中，可以用噪声来生成雾的图案。

```glsl
float getNoise(vec3 p){
    float noise=0.;
    noise=saturate(cnoise(p*vec3(.012,.012,0.)+vec3(iTime*.25))+.2);
    noise+=saturate(cnoise(p*vec3(.004,.004,0.)-vec3(iTime*.15))+.1);
    noise=saturate(noise);
    noise*=(1.-smoothstep(-5.,45.,p.y));
    noise*=(smoothstep(-200.,-35.,p.y));
    noise*=(smoothstep(0.,40.,p.x)+(1.-smoothstep(-40.,-0.,p.x)));
    return noise;
}
```

![0bb9024a-56c8-46fe-ace5-ba3aaaec45be](https://s2.loli.net/2023/10/16/FjYgNVbZ5OtRckW.png)

## 渲染优化

众神归位！目前场景是这样子的：

![8368d0a3-46cd-44c8-a8b0-9e4ddc96bf1d](https://s2.loli.net/2023/10/16/dj1pyE3qHFvDWX5.png)

可以看到后面的柱子太明显了，用`three.js`自带的雾[THREE.Fog](https://threejs.org/docs/#api/zh/scenes/Fog)来虚化它们。

![5ff7183e-b4a0-403a-a7ea-65def10bf5a0](https://s2.loli.net/2023/10/16/Eipza9ChDujYrK1.png)

添加后期处理效果，用到了[postprocessing](https://github.com/pmndrs/postprocessing)这个库。加上 2 个滤镜：辉光滤镜`BloomEffect`和色调映射滤镜`ToneMappingEffect`（色调映射为`ACES_FILMIC`）。

![7de9c17b-4e05-4d19-9317-131659d358a0](https://s2.loli.net/2023/10/16/NUy7WzGhrDYJQBp.png)

光这样还是不够。我们需要修改柱子的`Shader`，用`three.js`材质的`onBeforeCompile`方法可以基于材质本身的`Shader`来进行修改。

将原本的光照改成`Toon`风格的光照。

```glsl
float dotNL_toon=smoothstep(.25,.27,dotNL);
vec3 irradiance=dotNL_toon*directLight.color;
```

![198f92c5-8e5e-4b00-8bef-2b2760b32c26](https://s2.loli.net/2023/10/16/x9oHYOqlBEIAiv6.png)

Nice！这个渲染可以说是很还原了。

## 其他

至于路的动画、门的交互等，由于本身比较复杂，请自行查看项目源码，里面有对应的注释。

## 源码

Github 地址：https://github.com/alphardex/genshin-replica