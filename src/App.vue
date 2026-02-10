<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js'

const containerRef = ref<HTMLElement | null>(null)
let scene: THREE.Scene
let camera: THREE.PerspectiveCamera
let renderer: THREE.WebGLRenderer
let controls: OrbitControls
const meteorites: THREE.Mesh[] = []
let aurora: THREE.Mesh
let newYearText: THREE.Mesh | THREE.Group
let particles: THREE.Points
let animationState = 'meteorite' // meteorite, aurora, transition, newyear
let animationTime = 0

// 初始化场景
function initScene() {
  // 创建场景
  scene = new THREE.Scene()
  scene.background = new THREE.Color(0x000011)

  // 创建相机
  camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.z = 5

  // 创建渲染器
  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))

  if (containerRef.value) {
    containerRef.value.appendChild(renderer.domElement)
  }

  // 添加轨道控制器
  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.dampingFactor = 0.05
  controls.target.set(0, 0, 0) // 设置控制器目标为原点
  controls.update()

  // 创建流星雨
  createMeteorites()

  // 创建极光
  createAurora()

  // 创建3D新年快乐文字
  createNewYearText()

  // 创建粒子效果
  createParticles()

  // 监听窗口大小变化
  window.addEventListener('resize', onWindowResize)

  // 开始动画
  animate()
}

// 窗口大小变化处理
function onWindowResize() {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
  
  // 检测窗口是否达到最大尺寸
  checkWindowSize()
}

// 检测窗口大小，当达到最大时显示新年快乐文字
function checkWindowSize() {
  // 获取屏幕最大尺寸
  const screenWidth = window.screen.availWidth
  const screenHeight = window.screen.availHeight
  
  // 当窗口尺寸接近屏幕最大尺寸时显示文字
  if (window.innerWidth >= screenWidth * 0.95 && window.innerHeight >= screenHeight * 0.95) {
    if (newYearText && !newYearText.visible) {
      newYearText.visible = true
      particles.visible = true
      animationState = 'newyear'
    }
  }
}

// 创建流星雨
function createMeteorites() {
  // 使用锥体几何体创建光束效果
  const meteoriteGeometry = new THREE.ConeGeometry(0.01, 1, 8)
  
  for (let i = 0; i < 100; i++) { // 增加流星数量，实现全屏效果
    // 使用MeshLambertMaterial实现更好的光照效果
    const meteoriteMaterial = new THREE.MeshLambertMaterial({ 
      color: 0xffffff,
      transparent: true,
      opacity: 1
    })
    
    const meteorite = new THREE.Mesh(meteoriteGeometry, meteoriteMaterial)
    
    // 从右侧开始，全屏分布
    meteorite.position.x = 15 // 从右侧开始
    meteorite.position.y = (Math.random() - 0.5) * 20 // 全屏高度
    meteorite.position.z = (Math.random() - 0.5) * 10 // 前后分布
    
    // 旋转，使流星朝向左侧
    meteorite.rotation.x = Math.PI / 2 // 水平方向
    meteorite.rotation.y = Math.PI // 朝向左侧
    
    // 随机缩放
    meteorite.scale.setScalar(Math.random() * 0.8 + 0.5)
    
    // 添加速度和透明度属性
    ;(meteorite as any).speed = Math.random() * 0.5 + 0.3 // 更快的速度
    ;(meteorite as any).opacity = 1
    ;(meteorite as any).fading = false
    ;(meteorite as any).active = false // 初始状态为非活动
    ;(meteorite as any).delay = Math.random() * 5 // 随机延迟激活
    
    scene.add(meteorite)
    meteorites.push(meteorite)
  }
}

// 创建极光
function createAurora() {
  const auroraGeometry = new THREE.PlaneGeometry(20, 10, 100, 100)
  const auroraMaterial = new THREE.ShaderMaterial({
    uniforms: {
      time: { value: 0 },
      color: { value: new THREE.Color(0x00ffff) }
    },
    vertexShader: `
      uniform float time;
      varying vec2 vUv;
      void main() {
        vUv = uv;
        vec3 pos = position;
        pos.z += sin(pos.x * 0.5 + time) * 0.1;
        pos.z += sin(pos.y * 0.5 + time * 0.5) * 0.1;
        gl_Position = projectionMatrix * modelViewMatrix * vec4(pos, 1.0);
      }
    `,
    fragmentShader: `
      uniform float time;
      uniform vec3 color;
      varying vec2 vUv;
      void main() {
        float strength = sin(vUv.y * 10.0 + time) * 0.5 + 0.5;
        strength *= sin(vUv.x * 5.0 + time * 0.5) * 0.5 + 0.5;
        strength *= 0.5 + 0.5 * sin(time);
        gl_FragColor = vec4(color, strength * 0.3);
      }
    `,
    transparent: true,
    blending: THREE.AdditiveBlending
  })

  aurora = new THREE.Mesh(auroraGeometry, auroraMaterial)
  aurora.position.z = -2
  scene.add(aurora)
  aurora.visible = false
}

// 创建3D新年快乐文字
function createNewYearText() {
  // 使用CanvasTexture创建包含中文字符的纹理
  const canvas = document.createElement('canvas')
  canvas.width = 1024
  canvas.height = 256
  const context = canvas.getContext('2d')
  
  if (context) {
    // 设置字体和样式
    // 先填充黑色背景，确保文字可见
    context.fillStyle = '#000000'
    context.fillRect(0, 0, canvas.width, canvas.height)
    context.fillStyle = '#ff0000'
    context.font = 'bold 80px "Microsoft YaHei", "SimHei", sans-serif'
    context.textAlign = 'center'
    context.textBaseline = 'middle'
    
    // 绘制中文文本
    const text = '新年快乐 2026'
    context.fillText(text, canvas.width / 2, canvas.height / 2)
    
    // 创建纹理
    const texture = new THREE.CanvasTexture(canvas)
    texture.needsUpdate = true
    
    // 创建材质
    const textMaterial = new THREE.MeshPhongMaterial({ 
      map: texture,
      shininess: 100,
      specular: 0xffffff
    })
    
    // 创建一个平面几何体作为文字基础，使用合适的尺寸
    const planeGeometry = new THREE.PlaneGeometry(3, 0.8)
    
    // 创建3D文字组
    const textGroup = new THREE.Group()
    
    // 创建正面的平面，用于显示中文文本
    const frontMesh = new THREE.Mesh(planeGeometry, textMaterial)
    frontMesh.position.z = 0.1
    textGroup.add(frontMesh)
    
    // 创建背面的平面
    const backMesh = new THREE.Mesh(planeGeometry, textMaterial)
    backMesh.position.z = -0.1
    backMesh.rotation.y = Math.PI
    textGroup.add(backMesh)
    
    // 创建3D效果的侧面
    const sideGeometry = new THREE.BoxGeometry(3, 0.01, 0.2)
    const sideMaterial = new THREE.MeshPhongMaterial({ color: 0xff0000 })
    
    // 顶面
    const topMesh = new THREE.Mesh(sideGeometry, sideMaterial)
    topMesh.position.y = 0.4
    textGroup.add(topMesh)
    
    // 底面
    const bottomMesh = new THREE.Mesh(sideGeometry, sideMaterial)
    bottomMesh.position.y = -0.4
    textGroup.add(bottomMesh)
    
    // 左侧面
    const leftGeometry = new THREE.BoxGeometry(0.01, 0.8, 0.2)
    const leftMesh = new THREE.Mesh(leftGeometry, sideMaterial)
    leftMesh.position.x = -1.5
    textGroup.add(leftMesh)
    
    // 右侧面
    const rightMesh = new THREE.Mesh(leftGeometry, sideMaterial)
    rightMesh.position.x = 1.5
    textGroup.add(rightMesh)
    
    // 设置位置
    textGroup.position.set(0, 0, -5)
    scene.add(textGroup)
    textGroup.visible = false
    
    newYearText = textGroup
  }

  // 添加灯光
  const ambientLight = new THREE.AmbientLight(0x404040)
  scene.add(ambientLight)

  const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
  directionalLight.position.set(1, 1, 1)
  scene.add(directionalLight)

  const pointLight = new THREE.PointLight(0xff0000, 1, 100)
  pointLight.position.set(0, 0, 5)
  scene.add(pointLight)
}

// 创建粒子效果
function createParticles() {
  const particleGeometry = new THREE.BufferGeometry()
  const particleCount = 10000

  const positions = new Float32Array(particleCount * 3)
  const colors = new Float32Array(particleCount * 3)

  for (let i = 0; i < particleCount * 3; i += 3) {
    // 随机位置，限制在合理范围内
    positions[i] = (Math.random() - 0.5) * 20
    positions[i + 1] = (Math.random() - 0.5) * 20
    positions[i + 2] = (Math.random() - 0.5) * 10

    // 随机颜色
    colors[i] = Math.random() * 0.5 + 0.5 // 红色
    colors[i + 1] = Math.random() * 0.5 // 绿色
    colors[i + 2] = Math.random() * 0.5 // 蓝色
  }

  particleGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))
  particleGeometry.setAttribute('color', new THREE.BufferAttribute(colors, 3))

  const particleMaterial = new THREE.PointsMaterial({
    size: 0.03,
    vertexColors: true,
    blending: THREE.AdditiveBlending,
    transparent: true,
    opacity: 0.8
  })

  particles = new THREE.Points(particleGeometry, particleMaterial)
  scene.add(particles)
  particles.visible = false
}

// 动画循环
function animate() {
  requestAnimationFrame(animate)

  animationTime += 0.01

  // 更新轨道控制器
  controls.update()

  // 无论处于哪个状态，都更新流星，这样流星效果会一直存在
  updateMeteorites()

  // 检测窗口大小，当达到最大时显示新年快乐文字
  checkWindowSize()

  switch (animationState) {
    case 'meteorite':
      // 3秒后切换到极光效果
      if (animationTime > 3) {
        animationState = 'aurora'
        animationTime = 0
        aurora.visible = true
      }
      break

    case 'aurora':
      // 更新极光
      if (aurora.material instanceof THREE.ShaderMaterial && aurora.material.uniforms.time) {
        aurora.material.uniforms.time.value = animationTime
      }
      
      // 5秒后切换到穿越动画
      if (animationTime > 5) {
        animationState = 'transition'
        animationTime = 0
      }
      break

    case 'transition':
      // 穿越动画：相机快速移动
      camera.position.z -= 0.5
      
      // 2秒后切换到新年快乐效果
      if (animationTime > 2) {
        animationState = 'newyear'
        animationTime = 0
        camera.position.z = 5
        newYearText.visible = true
        particles.visible = true
      }
      break

    case 'newyear':
      // 更新3D文字旋转
      if (newYearText) {
        newYearText.rotation.y += 0.01
      }
      
      // 更新粒子效果
      if (particles) {
        particles.rotation.y += 0.005
      }
      break
  }

  // 渲染场景
  renderer.render(scene, camera)
}

// 更新流星雨
function updateMeteorites() {
  meteorites.forEach((meteorite) => {
    const m = meteorite as any
    
    // 处理延迟激活
    if (!m.active) {
      m.delay -= 0.01
      if (m.delay <= 0) {
        m.active = true
      }
      return
    }
    
    // 流星从右往左移动
    meteorite.position.x -= m.speed
    
    // 流星旋转
    meteorite.rotation.z += 0.01
    
    // 实现闪烁效果
    if (!m.fading) {
      // 随机开始淡出
      if (Math.random() < 0.02) {
        m.fading = true
      }
    } else {
      // 淡出效果
      m.opacity -= 0.05
      if (meteorite.material instanceof THREE.Material) {
        meteorite.material.opacity = m.opacity
      }
      
      // 完全淡出后重置
      if (m.opacity <= 0) {
        resetMeteorite(meteorite)
      }
    }
    
    // 流星超出边界后重置
    if (meteorite.position.x < -15) {
      resetMeteorite(meteorite)
    }
  })
}

// 重置流星
function resetMeteorite(meteorite: THREE.Mesh) {
  const m = meteorite as any
  
  // 重置位置（从右侧重新开始）
  meteorite.position.x = 15
  meteorite.position.y = (Math.random() - 0.5) * 20 // 全屏高度
  meteorite.position.z = (Math.random() - 0.5) * 10 // 前后分布
  
  // 重置旋转
  meteorite.rotation.x = Math.PI / 2 // 水平方向
  meteorite.rotation.y = Math.PI // 朝向左侧
  meteorite.rotation.z = 0
  
  // 重置透明度和状态
  m.opacity = 1
  m.fading = false
  m.active = false // 重置为非活动状态，实现间断出现
  m.delay = Math.random() * 3 // 随机延迟，实现间断效果
  if (meteorite.material instanceof THREE.Material) {
    meteorite.material.opacity = 1
  }
  
  // 重置速度
  m.speed = Math.random() * 0.5 + 0.3
}

// 挂载时初始化
onMounted(() => {
  initScene()
})

// 卸载时清理
onUnmounted(() => {
  window.removeEventListener('resize', onWindowResize)
  if (containerRef.value && renderer.domElement) {
    containerRef.value.removeChild(renderer.domElement)
  }
  renderer.dispose()
  controls.dispose()
})
</script>

<template>
  <div ref="containerRef" class="container"></div>
</template>

<style scoped>
.container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}
</style>
