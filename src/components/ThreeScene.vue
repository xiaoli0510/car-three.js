<script setup lang='ts'>
import { GLTFLoader } from "three/addons/loaders/GLTFLoader.js";
// 引入轨道控制器扩展库OrbitControls.js
import { OrbitControls } from 'three/addons/controls/OrbitControls.js'
import * as THREE from "three";
import { onMounted, ref } from "vue";

const container = ref()
let carBody = ref('')
const initScene = () => {

    const scene = new THREE.Scene();
    scene.background = new THREE.Color('#cccccc')
    const camera = new THREE.PerspectiveCamera(
        75,
        window.innerWidth / window.innerHeight,
        0.1,
        1000
    );

    const renderer = new THREE.WebGLRenderer();
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setAnimationLoop(animate);
    container.value.appendChild(renderer.domElement);

    const loader = new GLTFLoader();
    let model;
    loader.load(
        '/src/assets/models/red_car.glb',
        function (gltf) {
            model = gltf.scene
            model.scale.set(0.02, 0.02, 0.02)
            scene.add(model);

            gltf.animations; // Array<THREE.AnimationClip>
            gltf.scene; // THREE.Group
            gltf.scenes; // Array<THREE.Group>
            gltf.cameras; // Array<THREE.Camera>
            gltf.asset; // Object

            model.traverse(child => {
                console.log(child.name)

                // 手动指定车身部件的确切名称
                const bodyPartNames = [
                    // 'Object_2',  // 根据你之前的结构，这些可能是车身
                    'Object_3',
                    // 添加其他车身部件的确切名称
                ]

                const isBodyPart = (meshName: string): boolean => {
                    return bodyPartNames.includes(meshName)
                }
                // 找到名为 'carobjcleanermaterialmergergles' 的车身对象
                if (isBodyPart(child.name)) {
                    carBody.value = child
                    console.log('找到车身:', carBody.value)

                    // 打印车身的所有子网格（部件）
                    child.children.forEach((mesh, index) => {
                        if (mesh instanceof THREE.Mesh) {
                            console.log(`车身部件 ${index + 1}:`, mesh.name)
                            console.log('材质:', mesh.material)
                        }
                    })
                }
            })

        },
        // called while loading is progressing
        function (xhr) {
            console.log((xhr.loaded / xhr.total) * 100 + "% loaded");
        },
        // called when loading has errors
        function (error) {
            console.log("An error happened");
        }
    );

    // 环境光
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
    scene.add(ambientLight);

    // 定向光
    const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
    directionalLight.position.set(10, 10, 5);
    scene.add(directionalLight);

    camera.position.z = 5;

    function animate() {
        //   scene.rotateX += 1;
        //   requestAnimationFrame(animate);

        //   if (sprite) {
        //    sprite.material.rotation += 10;
        //   }

        renderer.render(scene, camera);
    }

    // 设置相机控件轨道控制器OrbitControls
    const controls = new OrbitControls(camera, renderer.domElement)

    // 监听轨道控制器的change事件，监听到改变时，重新执行渲染操作渲染三维场景
    controls.addEventListener('change', function () {
        renderer.render(scene, camera)
    })
}

onMounted(() => {
    initScene()
})

const colors = [
    { value: '#000', name: '黑色' },
    { value: '#fff', name: '白色' },
    { value: '#1B6EF3', name: '蓝色' },
    { value: '#41B883', name: '蓝色' },
]
const changeColor = (color) => {
    if (!carBody.value) {
        console.log('车身尚未加载完成')
        return
    }

    console.log('修改颜色为:', color)

    // 遍历车身的所有子网格并修改颜色
    carBody.value.traverse((child) => {
        if (child instanceof THREE.Mesh && child.material) {
            // 确保材质有 color 属性
            if (child.material instanceof THREE.MeshBasicMaterial ||
                child.material instanceof THREE.MeshStandardMaterial ||
                child.material instanceof THREE.MeshPhongMaterial) {

                child.material.color.set(color)

                // 如果需要，可以同时修改其他材质属性
                child.material.needsUpdate = true
            }
        }
    })
}


</script>
<template>
    <div v-for="(item, index) in colors" :key="index" @click="changeColor(item.value)">{{ item.name }}</div>
    <div ref="container" class="three-container"></div>
</template>