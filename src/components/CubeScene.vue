<script setup lang='ts'>
import { GLTFLoader } from "three/addons/loaders/GLTFLoader.js";
// 引入轨道控制器扩展库OrbitControls.js
import { OrbitControls } from 'three/addons/controls/OrbitControls.js'
import * as THREE from "three";
import { onMounted, ref, render } from "vue";
import { DRACOLoader } from "three/examples/jsm/Addons.js";

const container = ref()
const initScene = () => {
    //创建场景
    const scene = new THREE.Scene();
    //创建camera
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)

    //创建渲染器
    const renderer = new THREE.WebGLRenderer()
    //设置渲染器的大小
    renderer.setSize(window.innerWidth, window.innerHeight)
    renderer.setClearColor(0x87CEEB)
    //将渲染器append
    container.value.appendChild(renderer.domElement)

    camera.position.set(0, 500, 150)



  const textLoader = new THREE.TextureLoader().load('/src/assets/imgs/grass.jpg')
  scene.background = textLoader


    const loader = new GLTFLoader()
    let model
    loader.load('/src/assets/models/red_car.glb', (gltf) => {
        console.log('gltf', gltf);
        model = gltf.scene

        scene.add(model)
    }, function (xhr) {

        console.log((xhr.loaded / xhr.total * 100) + '% loaded');

    },
        // called when loading has errors
        function (error) {

            console.log('An error happened');

        })

    const directionalLight = new THREE.DirectionalLight(0xffffff, 0.9);
    scene.add(directionalLight)



    //添加轨道控制器去用camera控制ring

    const controls = new OrbitControls(camera, renderer.domElement)
    controls.update()


    function animate() {
        // ring.rotation.x += 0.01;
        // ring.rotation.y += 0.01;
        controls.update()

        renderer.render(scene, camera)
    }

    renderer.setAnimationLoop(animate)
}


onMounted(() => {
    initScene()
})


</script>
<template>
    <!-- <div v-for="(item, index) in colors" :key="index" @click="changeColor(item.value)">{{ item.name }}</div> -->
    <div ref="container" class="three-container"></div>
</template>