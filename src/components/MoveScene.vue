<script setup lang='ts'>
import { GLTFLoader } from "three/addons/loaders/GLTFLoader.js";
import { OrbitControls } from 'three/addons/controls/OrbitControls.js'
import * as THREE from "three";
import { onMounted, ref  } from "vue";

const container = ref()
let carBody;
//网格大小
const gridSize = 1000;
const initScene = () => {
    //创建场景
    const scene = new THREE.Scene();
    //创建camera
    const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)

    //创建渲染器
    const renderer = new THREE.WebGLRenderer({antialias: true})
    //设置渲染器的大小
    renderer.setSize(window.innerWidth / 2, window.innerHeight / 2)
    renderer.setClearColor(0xffffff)

    // Grid
    const divisions = 50; //网格被分割成多少份
    const gridHelper = new THREE.GridHelper(gridSize, divisions);
    scene.add(gridHelper);

    //  camera.position.set(0, 500, 150)
    camera.position.set(0, 500, 150);

    //将渲染器append
    container.value.appendChild(renderer.domElement)


    const loader = new GLTFLoader()
    let model
    loader.load('/src/assets/models/red_car.glb', (gltf) => {
        console.log('gltf', gltf);
        model = gltf.scene
        model.scale.set(3, 3, 3)
        // model旋转 90°
        model.rotation.y = Math.PI / 2
        scene.add(model)
        carBody = model.getObjectByName('carobjcleanermaterialmergergles')
    }, function (xhr) {

    },
        function (error) {
            console.log('An error happened');
        })
    const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
    scene.add(directionalLight)

    //轨道
    const controls = new OrbitControls(camera, renderer.domElement)
    controls.update()

    function animate() {
        controls.update()
        gridHelper.position.x += 5
        //当网格移出屏幕时，让网格重新出现做屏幕左侧，实现无线循环
        if (gridHelper.position.x > gridSize) {
            gridHelper.position.x = -gridSize
        }
        renderer.render(scene, camera)
    }

    renderer.setAnimationLoop(animate)
}


onMounted(() => {
    initScene()
})


const colors = [
    { value: '#000', name: '黑色' },
    { value: '#fff', name: '白色' },
    { value: '#1B6EF3', name: '蓝色' },
    { value: '#41B883', name: '绿色' },
]
const changeColor = (color) => {
    if (!carBody) return;
    carBody.traverse((child) => {
        if (child.isMesh && child.material) {
            // 创建新材质或修改现有材质
            if (Array.isArray(child.material)) {
                // 处理材质数组
                child.material.forEach(material => {
                    if (material instanceof THREE.MeshStandardMaterial) {
                        material.color.set(color);
                        material.needsUpdate = true;
                    }
                });
            } else if (child.material.color) {
                //如果有贴图，那么修改颜色只会修改透明部分，所以需要为null
                if (child.material.map) {
                    child.material.map = null;
                }
                // 处理单个材质
                child.material.color.set(color);
                child.material.needsUpdate = true;
            }
        }
    });
}

</script>
<template>
    <div class="car-wrap">
        <div class="btn-color" v-for="(item, index) in colors" :key="index" @click="changeColor(item.value)"
            :style="{ 'backgroundColor': item.value }"></div>
        <div ref="container" class="three-container"></div>
    </div>
</template>
<style scoped>
.car-wrap {
    background: #ccc;
}

.btn-color {
    width: 50px;
    height: 30px;
    margin: 10px auto;
}
</style>