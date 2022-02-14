<script lang="ts" setup>
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { onMounted } from 'vue';
const earth_bg = './src/assets/earth_bg.png';
const sky = './src/assets/sky.png';
const star1 = './src/assets/starflake1.png';
const star2 = './src/assets/starflake2.png';
const clouds = './src/assets/cloud.png';

let container: any
let width: number
let half_width: number
let height: number
let distance: number
let scene: any
let camera: any
let renderer: any
let controls: any
let boxMesh: any
let earthGroup: any
let starGroup: any
let cloudsArray: any[] = []

const createScene = () => {
    scene = new THREE.Scene();
    // scene.fog = new THREE.Fog(0x000000, 0, 10000)
}

const createBox = () => {
    // TextureLoader创建一个纹理加载器对象，可以加载图片作为几何体纹理
    new THREE.TextureLoader().load(sky, (texture) => {
        const geometry = new THREE.SphereGeometry(half_width, 100, 100);
        const material = new THREE.MeshBasicMaterial({ map: texture, side: THREE.BackSide});
        boxMesh = new THREE.Mesh( geometry, material );
        scene.add( boxMesh );
    });
}

const createCamera = () => {
    distance = Math.floor(0.5 * Math.sqrt(width * width + height * height) / Math.tan(Math.PI / 4));
    camera = camera = new THREE.PerspectiveCamera(45, width / height, 1, 30000);
    camera.position.set(-distance, 0, 0); //设置相机位置
    camera.lookAt(scene.position); //设置相机方向(指向的场景对象)
}

const createLight = () => {
    //环境光
    const ambient = new THREE.AmbientLight(0xffffff, 1);
    scene.add(ambient);

    //点光源
    const point = new THREE.PointLight(0x0655fd, 5, 0); // 颜色、强度、距离
    point.position.set(0, -200, 300); //点光源位置
    scene.add(point); //点光源添加到场景中
}

const createHelper = () => {
    const axisHelper = new THREE.AxesHelper(300);
    scene.add(axisHelper);
}

const createEarth = () => {
    new THREE.TextureLoader().load(earth_bg, (texture) => {
        const geometry = new THREE.SphereGeometry(60, 50, 50); // 创建一个球体几何对象
        const material = new THREE.MeshPhongMaterial({ map: texture }); // 创建地球贴图材质
        const mesh = new THREE.Mesh( geometry, material );
        earthGroup = new THREE.Group();
        earthGroup.add(mesh);
        earthGroup.position.x = 100;
        earthGroup.position.y = 250;
        earthGroup.position.z = -450;
        scene.add( earthGroup );
    });
}

const createStar = () => {
    const texture1 = new THREE.TextureLoader().load(star1);
    const texture2 = new THREE.TextureLoader().load(star2);
    
    starGroup = new THREE.Group(); 

    for(let i = 0; i < 500; i++){
        const value = Math.random() * 25,
            x = Math.random() * half_width * (Math.random() > 0.5 ? 1 : -1),
            y = Math.random() * half_width * (Math.random() > 0.5 ? 1 : -1),
            z = Math.random() * half_width * (Math.random() > 0.5 ? 1 : -1);
        // 精灵材质
        const spriteMaterial = new THREE.SpriteMaterial({
            map: Math.random() > 0.5 ? texture1 : texture2, // 随机设置精灵纹理贴图
            transparent: true,
            opacity: 1,
        });
        // 创建精灵模型对象
        const sprite = new THREE.Sprite(spriteMaterial);
        sprite.scale.set(value, value, 1);
        sprite.position.set(x, y, z)

        starGroup.add(sprite);
    }
    scene.add(starGroup);
}

const createPathAndCloud = (points: any, w: number, h: number) => {
    const curve = new THREE.CatmullRomCurve3(points)
    // 查看浮云路径
    // const tubeGeometry = new THREE.TubeGeometry(curve, 100, 5, 50, false)
    // const tubeMaterial = new THREE.MeshBasicMaterial({
    //     color: '0xffffff',
    //     opacity: 1,
    //     transparent: true
    // })
    // const tube = new THREE.Mesh(tubeGeometry, tubeMaterial)
    // scene.add(tube)

    const clondGeometry = new THREE.PlaneGeometry(w, h)
    const cloudTexture = new THREE.TextureLoader().load(clouds)
    const clondMaterial = new THREE.MeshBasicMaterial({
        map: cloudTexture,
        side: THREE.DoubleSide,
        transparent: true,
    })
    const cloud = new THREE.Mesh(clondGeometry, clondMaterial)
    cloud.rotateY(-Math.PI / 2)
    scene.add(cloud)

    return {
        cloud,
        curve
    }
}

const getCloudMoveFun = (
    cloudParameter: any,
    speed = 0.0005,
    scaleSpeed = 0.0005,
    maxScale = 1,
    startScale = 0
) => {
    let cloudProgress = 0
    return () => {
        if (cloudProgress > 1) {
        cloudProgress = 0
        startScale = 0
        } else {
            if (startScale < maxScale) {
                startScale += scaleSpeed
                cloudParameter.cloud.scale.setScalar(startScale)
            }
            cloudProgress += speed
            const point = cloudParameter.curve.getPoint(cloudProgress)
            cloudParameter.cloud.position.set(point.x, point.y, point.z)
        }
    }
}

const createClouds = () => {
    const cloud1 = createPathAndCloud(
        [
            new THREE.Vector3(200, 200, -300),
            new THREE.Vector3(50, 100, -100),
            new THREE.Vector3(-400, 120, -400)
        ],
        400,
        200
    )
    const move1 = getCloudMoveFun(cloud1)
    cloudsArray.push(move1)

    const cloud2 = createPathAndCloud(
        [
            new THREE.Vector3(500, -100, 100),
            new THREE.Vector3(0, 200, 200),
            new THREE.Vector3(200, 420, 400)
        ],
        500,
        250
    )
    const move2 = getCloudMoveFun(cloud2, 0.0008, 0.0008)
    cloudsArray.push(move2)
}

const init = ()=>{
    /**
     * 创建场景对象Scene
     */
    createScene();

    /**
     * 创建盒模型
     */
    createBox();
    
    /**
     * 相机设置
     */
    createCamera();

    /**
     * 光源设置
     */
    createLight();

    /**
     * 辅助坐标轴
     */
    // createHelper();

    /**
     * 创建地球
     */
    createEarth();

    /**
     * 创建星星
     */
    createStar();

    /**
     * 创建浮云
     */
    createClouds();

    /**
     * 创建渲染器对象
     */
    renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
    renderer.setSize(width, height);
    renderer.shadowMap.enabled = true; // 使用阴影贴图

    function render() {
        renderer.render(scene,camera);

        if(earthGroup) earthGroup.rotateY(0.002);

        starGroup.children.forEach(sprite => {
            sprite.position.x -= 1;
            if (sprite.position.x < (-half_width)) {
                sprite.position.x = half_width;
            }
        });

        cloudsArray.forEach(move => {
            move()
        })

        requestAnimationFrame(render);
    }
    render()
    controls = new OrbitControls(camera,renderer.domElement);//创建控件对象
    return renderer.domElement
}

onMounted(() => {
    container = document.getElementById('stage')
    width = container.clientWidth
    half_width = width / 2
    height = container.clientHeight

    container.appendChild(init())
})

</script>

<template>
  <div class="loginStage">
    <div class="stage" id="stage"></div>
    <div class="ground"></div>
    <div class="human"></div>
  </div>
</template>

<style lang="less" scoped>
.loginStage {
    width: 100%;
    height: 100%;
    .stage {
        width: 100%;
        height: 100%;
    }
    .ground {
        position: absolute;
        top: 50%;
        width: 100%;
        height: 50%;
        background-image: url('./src/assets/ground.png');
    }
    @keyframes humanMove {
        0% {
            top: 30%;
        }
        25% {
            top: 40%;
        }
        50% {
            top: 30%;
        }
        75% {
            top: 20%;
        }
        100% {
            top: 30%;
        }
    }
    .human {
        position: absolute;
        top: 30%;
        left: 45%;
        width: 308px;
        height: 380px;
        background-image: url('./src/assets/login_human.png');
        animation: humanMove 8s linear 0s infinite normal none;
    }
}
</style>
