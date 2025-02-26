# eye-floaters-simulator
在浏览器页面模拟飞蚊症的效果，by Claude 3.7 Sonnet

# 改进后的飞蚊症模拟器 Tampermonkey 脚本

以下是改进版的脚本，添加了边缘模糊渐变效果，让飞蚊看起来更加真实自然：

```javascript
// ==UserScript==
// @name         飞蚊症模拟器 (带模糊边缘)
// @namespace    http://tampermonkey.net/
// @version      0.2
// @description  模拟飞蚊症效果，在屏幕上显示带模糊边缘的灰色半透明漂浮物
// @author       AI助手
// @match        *://*/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';
  
    // 创建一个容器来放置所有飞蚊
    const container = document.createElement('div');
    container.style.position = 'fixed';
    container.style.top = '0';
    container.style.left = '0';
    container.style.width = '100%';
    container.style.height = '100%';
    container.style.pointerEvents = 'none'; // 确保不会干扰页面交互
    container.style.zIndex = '9999';
    document.body.appendChild(container);
  
    // 飞蚊的数量
    const floaterCount = 15;
  
    // 创建飞蚊对象数组
    const floaters = [];
  
    // 创建飞蚊元素
    for (let i = 0; i < floaterCount; i++) {
        createFloater();
    }
  
    // 创建一个飞蚊元素
    function createFloater() {
        // 随机基础大小 (3-10px)
        const baseSize = 3 + Math.random() * 7;
        // 实际大小 = 基础大小 + 模糊边缘
        const blurRadius = baseSize * 1.5;
        const size = baseSize + blurRadius * 2;
      
        // 创建飞蚊元素
        const floater = document.createElement('div');
        floater.style.position = 'absolute';
        floater.style.width = `${baseSize}px`;
        floater.style.height = `${baseSize}px`;
        floater.style.borderRadius = '50%';
      
        // 使用多种样式组合创建模糊的边缘效果
        floater.style.backgroundColor = 'rgba(100, 100, 100, 0.3)';
        floater.style.boxShadow = `0 0 ${blurRadius}px ${blurRadius/2}px rgba(100, 100, 100, 0.2)`;
        floater.style.filter = `blur(${blurRadius/3}px)`;
      
        // 随机形状变化 (有些飞蚊是线状或不规则形状)
        if (Math.random() > 0.7) {
            floater.style.width = `${baseSize * (1 + Math.random())}px`;
            floater.style.height = `${baseSize * 0.5}px`;
            floater.style.transform = `rotate(${Math.random() * 360}deg)`;
        }
      
        // 随机初始位置
        const x = Math.random() * window.innerWidth;
        const y = Math.random() * window.innerHeight;
      
        // 随机速度 (较大的飞蚊移动较慢)
        const speedFactor = 1 - (baseSize / 15); // 越大越慢
        const speedX = (Math.random() - 0.5) * 2 * speedFactor;
        const speedY = (Math.random() - 0.5) * 2 * speedFactor;
      
        // 将飞蚊添加到容器
        container.appendChild(floater);
      
        // 存储飞蚊信息
        floaters.push({
            element: floater,
            x: x,
            y: y,
            speedX: speedX,
            speedY: speedY,
            baseSize: baseSize,
            totalSize: size,
            rotationSpeed: (Math.random() - 0.5) * 0.5, // 旋转速度
            rotation: Math.random() * 360, // 初始旋转角度
            opacity: 0.2 + Math.random() * 0.3 // 随机透明度
        });
      
        // 设置初始位置
        floater.style.left = `${x - size/2}px`;
        floater.style.top = `${y - size/2}px`;
    }
  
    // 动画函数
    function animate() {
        // 更新每个飞蚊的位置
        floaters.forEach(floater => {
            // 随机微调速度（使运动更自然）
            floater.speedX += (Math.random() - 0.5) * 0.05;
            floater.speedY += (Math.random() - 0.5) * 0.05;
          
            // 限制最大速度
            const maxSpeed = 1.5;
            floater.speedX = Math.max(-maxSpeed, Math.min(maxSpeed, floater.speedX));
            floater.speedY = Math.max(-maxSpeed, Math.min(maxSpeed, floater.speedY));
          
            // 更新位置
            floater.x += floater.speedX;
            floater.y += floater.speedY;
          
            // 边界检测 (考虑到飞蚊的大小)
            const margin = floater.totalSize;
            if (floater.x < -margin || floater.x > window.innerWidth + margin) {
                floater.x = (floater.x < 0) ? window.innerWidth + margin : -margin;
            }
            if (floater.y < -margin || floater.y > window.innerHeight + margin) {
                floater.y = (floater.y < 0) ? window.innerHeight + margin : -margin;
            }
          
            // 更新旋转
            floater.rotation += floater.rotationSpeed;
          
            // 随机轻微改变透明度
            floater.opacity += (Math.random() - 0.5) * 0.01;
            floater.opacity = Math.max(0.1, Math.min(0.5, floater.opacity));
          
            // 应用新位置、旋转和透明度
            floater.element.style.left = `${floater.x - floater.totalSize/2}px`;
            floater.element.style.top = `${floater.y - floater.totalSize/2}px`;
            floater.element.style.transform = `rotate(${floater.rotation}deg)`;
            floater.element.style.opacity = floater.opacity;
        });
      
        // 继续动画
        requestAnimationFrame(animate);
    }
  
    // 开始动画
    animate();
  
    // 跟随视线移动效果 (当用户移动鼠标时，飞蚊也会轻微移动)
    document.addEventListener('mousemove', function(e) {
        const mouseX = e.clientX;
        const mouseY = e.clientY;
      
        floaters.forEach(floater => {
            // 计算距离
            const dx = mouseX - floater.x;
            const dy = mouseY - floater.y;
            const distance = Math.sqrt(dx*dx + dy*dy);
          
            // 距离越近影响越大
            if (distance < 300) {
                const influence = (300 - distance) / 10000;
                floater.speedX -= dx * influence;
                floater.speedY -= dy * influence;
            }
        });
    });
  
    // 窗口大小改变时调整
    window.addEventListener('resize', function() {
        floaters.forEach(floater => {
            // 如果飞蚊超出了新的窗口大小，将其放回窗口内
            if (floater.x > window.innerWidth) {
                floater.x = window.innerWidth - 10;
            }
            if (floater.y > window.innerHeight) {
                floater.y = window.innerHeight - 10;
            }
        });
    });
})();
```

## 改进内容

1. **模糊边缘效果**：
   - 使用 `boxShadow` 创建向外扩散的模糊效果
   - 添加 `filter: blur()` 使整个飞蚊的边缘更加柔和
   - 结合透明度实现渐变效果

2. **更多真实细节**：
   - 不同形状的飞蚊（圆形和线形）
   - 飞蚊大小与移动速度关联（较大的飞蚊移动较慢）
   - 添加旋转动画
   - 动态变化的透明度

3. **视线跟随效果**：
   - 当鼠标移动时，飞蚊会轻微受到影响，模拟眼球转动时飞蚊的漂移

## 使用说明

1. 安装 Tampermonkey 浏览器扩展
2. 点击 Tampermonkey 图标，选择"添加新脚本"
3. 删除编辑器中的默认代码，粘贴上面的脚本代码
4. 点击"保存"按钮

现在，当你浏览网页时，会看到更加逼真的飞蚊症模拟效果，带有模糊渐变的边缘，让飞蚊看起来更加自然。

你可以根据需要调整脚本中的参数，如飞蚊数量、大小、模糊程度等，以获得最符合你体验的效果。
