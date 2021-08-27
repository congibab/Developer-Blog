---
title:  "three-glsl"
excerpt: "three-glsl"

categories:
  - Blog
tags:
  - Blog
last_modified_at: 2021-08-28
classes: wide
---

* [プログラムへ移動(実行)](https://congibab.github.io/threejs-glsl/)

<!-- ## Local上に実行する場合
``` bash
npm install
npm run build
``` -->

<img src = "{{ site.url }}{{ site.baseurl }}/assets/gifs/threejs-glsl.gif">

* [GitHubへ移動](https://github.com/congibab/threejs-glsl)

## 概要
* 制作期間 : 2021.08.16 ~ 研究中
* 制作意図 : ウェブ上にShaderを公開したいと思いました。
* 制作人数 : 個人
* プラットフォーム : ウェブブラウザ(Chrome,Firefox,Safari)
* 使用言語 : TypeScript, GLSL
* 使用ライブラリ : webpack, Three.js

## 操作方法
* マウス左クリック：カメラ回転
* マウス右クリック：カメラ移動
* マウスホイール：ズームアウト, ズームイン

## アピールポイント
* ゲームエフェクト製作に興味があって、Shaderを研究しています。
* 3Dパソコングラフィックス知識を学んでいます。
* WebPackを利用して、ShaderFileをモジュール化ができます。
* TypeScriptの使い方を学びました。
* ShaderFile(GLSL)をUnityShaderで変換できます。

## ソースコード(簡略化)
### index.ts
```ts
import * as THREE from 'three'

import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';

import Fresnel_vShader from './glsl/Fresnel.vert';
import Fresnel_fShader from './glsl/Fresnel.frag';

let Time: number;
let deltaTime: number;

//初期化
window.addEventListener("DOMContentLoaded", () => {

  const testManager = Test.getInstance();
  testManager.view();

  const clock = new THREE.Clock();
  Time = clock.getElapsedTime();
  deltaTime = clock.getDelta();

  const stats = Stats();
  document.body.appendChild(stats.dom)

  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);

  const renderer = new THREE.WebGLRenderer();
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.body.appendChild(renderer.domElement);

  const controls = new OrbitControls(camera, renderer.domElement);
  
  //Grid描画
  const gridHelper = new THREE.GridHelper(100, 100);
  scene.add(gridHelper);

  const axesHelper = new THREE.AxesHelper(5);
  scene.add(axesHelper);

  const texture1 = new THREE.TextureLoader().load("/dist/asset/textures/HexPulse.png");
  const geometry = new THREE.SphereGeometry();
  
  //Shaderへ渡す変数
  const uniforms = {
    "texture1": { value: texture1 },
    "time": { value: Time },
  }

  const Fresnel_material = new THREE.ShaderMaterial({
    uniforms: uniforms,
    transparent: true,
    vertexShader: Fresnel_vShader,
    fragmentShader: Fresnel_fShader,
  });

  //shpere追加
  const shpere = new THREE.Mesh(geometry, Fresnel_material);
  scene.add(shpere);
  shpere.position.y = 1.0;

  camera.position.z = 5;
  camera.position.y = 10;

  //画面のサイズが変更すると、サイズに合わせて描画
  window.addEventListener('resize', onWindowResize, false);
  function onWindowResize() {

    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  }

  //Update(更新)
  function animate() {
    requestAnimationFrame(animate);

    Time = clock.getElapsedTime();
    deltaTime = clock.getDelta();
    uniforms['time'].value = Time;

    stats.update();
    controls.update();
    renderer.render(scene, camera);
  };

  animate();
})
```


### dissolve.vert
```glsl
varying vec2 vUv;
varying vec3 vNormal;

varying vec3 ViewDir;
uniform float time;

void main()
{
  vUv = uv;
  vNormal = normal;
  
  vec4 worldPostion = modelMatrix * vec4(position, 1.0 );
  ViewDir = cameraPosition - worldPostion.xyz;
  
  gl_Position = projectionMatrix*modelViewMatrix* vec4( position , 1.0 );
}

```

### dissolve.frag
``` glsl
precision mediump float;
precision mediump int;

varying vec2 vUv;
varying vec3 vNormal;
varying vec3 ViewDir;

uniform sampler2D texture1;
uniform float time;

float Fresnel(vec3 Normal,vec3 ViewDir,float Power)
{
    
    float test2=dot(normalize(ViewDir),normalize(Normal));
    float test1=clamp(test2,0.,1.);
    
    return pow(1.-test1,Power);
}

void main()
{
    float _FresnelEffect=Fresnel(vNormal,ViewDir,2.);
    vec3 color = vec3(0.1098, 0.6157, 0.8118);
    gl_FragColor=vec4(color,_FresnelEffect)*texture2D(texture1,fract(vUv*2.));
}
```

## 参考サイト
[Unity Shader Gtaph Document](https://docs.unity3d.com/Packages/com.unity.shadergraph@6.9/manual/Node-Library.html)

[Three.js](https://threejs.org/docs/index.html#manual/en/introduction/Creating-a-scene)