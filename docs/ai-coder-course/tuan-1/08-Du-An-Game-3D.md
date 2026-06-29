# 🎮 Bài 8: Dự Án — Xây Dựng Game 3D với Cursor AI

> Thực hành Vibe Coding: Tạo game 3D hoàn chỉnh chỉ bằng prompt

---

## 1. Mục Tiêu Dự Án

Trong khóa học gốc, học viên xây dựng một **game 3D** hoàn chỉnh chỉ bằng cách nói chuyện với Cursor AI. Đây là minh chứng mạnh mẽ nhất cho sức mạnh của Vibe Coding.

### Công nghệ sử dụng:
- **Three.js** — Thư viện 3D cho web
- **HTML/CSS/JavaScript** — Frontend
- **Cursor Agent Mode** — AI viết toàn bộ code

---

## 2. Quy Trình Vibe Coding Game

### Bước 1: Mô tả ý tưởng game

```
Prompt ban đầu:
"Tạo một game 3D chạy trên browser bằng Three.js:

- Nhân vật chính: khối lập phương có thể di chuyển bằng WASD
- Môi trường: mặt phẳng với cỏ xanh, bầu trời gradient
- Chướng ngại vật: các cột trụ ngẫu nhiên phải né tránh
- Mục tiêu: thu thập các ngôi sao vàng rải trên map
- Hiệu ứng: ánh sáng, bóng đổ, particle khi nhặt sao
- HUD: điểm số, thời gian chơi"
```

### Bước 2: AI sinh code cơ bản → Review

```
Cursor Agent tạo:
📁 game/
├── index.html      ← Cấu trúc HTML
├── style.css       ← Styling HUD
├── main.js         ← Game logic chính
├── player.js       ← Điều khiển nhân vật
├── obstacles.js    ← Logic chướng ngại vật
└── particles.js    ← Hiệu ứng particle
```

### Bước 3: Iterate & Polish

```
"Thêm hiệu ứng:

- Camera follow player mượt mà (lerp)
- Particle explosion khi nhặt sao
- Âm thanh coin pickup (Web Audio API)
- Menu start/pause/game over
- Bảng high score lưu localStorage"
```

### Bước 4: Debug bằng Vibe Coding

```
"Game bị lag khi có nhiều particle. Tối ưu bằng object pooling
và giới hạn số particle tối đa 100"
```

---

## 3. Ví Dụ Code (AI sinh ra)

```javascript
// main.js - Cursor Agent Mode tạo
import * as THREE from 'three';

class Game {
  constructor() {
    this.scene = new THREE.Scene();
    this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
    this.renderer = new THREE.WebGLRenderer({ antialias: true });
    
    this.score = 0;
    this.stars = [];
    this.obstacles = [];
    
    this.init();
  }

  init() {
    // Setup renderer
    this.renderer.setSize(window.innerWidth, window.innerHeight);
    this.renderer.shadowMap.enabled = true;
    document.body.appendChild(this.renderer.domElement);

    // Sky gradient
    this.scene.background = new THREE.Color(0x87CEEB);
    
    // Ground
    const ground = new THREE.Mesh(
      new THREE.PlaneGeometry(100, 100),
      new THREE.MeshStandardMaterial({ color: 0x4CAF50 })
    );
    ground.rotation.x = -Math.PI / 2;
    ground.receiveShadow = true;
    this.scene.add(ground);

    // Player (cube)
    this.player = new THREE.Mesh(
      new THREE.BoxGeometry(1, 1, 1),
      new THREE.MeshStandardMaterial({ color: 0x2196F3 })
    );
    this.player.position.y = 0.5;
    this.player.castShadow = true;
    this.scene.add(this.player);

    // Lights
    const sun = new THREE.DirectionalLight(0xffffff, 1);
    sun.position.set(10, 20, 10);
    sun.castShadow = true;
    this.scene.add(sun);

    // Spawn stars and obstacles
    this.spawnStars(20);
    this.spawnObstacles(15);
    
    this.animate();
  }

  // ... AI tiếp tục sinh code cho movement, collision, scoring
}

new Game();
```

---

## 4. Bài Học Rút Ra

### ✅ Vibe Coding hiệu quả khi:
- Prototype nhanh ý tưởng
- Dự án cá nhân, không cần production-ready
- Học công nghệ mới (Three.js) mà không biết sâu
- Iterate nhanh trên UI/UX

### ⚠️ Giới hạn phát hiện:
- AI có thể tạo code không tối ưu (performance)
- Logic phức tạp (physics, AI pathfinding) cần human intervention
- Code structure có thể messy nếu không có rules file
- Khó scale lên dự án lớn nếu không có engineering discipline

---

## 5. Thử Thách Cho Bạn

Mở Cursor hoặc Antigravity và thử tạo:

1. **Game Snake** 2D với scoreboard
2. **Game Memory Card** với animation flip
3. **Game Platformer** đơn giản với Three.js

Chỉ dùng prompt, **không gõ code thủ công**!

---

> **Tiếp theo**: [Bài 9: Website cá nhân AI Twin →](09-Du-An-Website-AI-Twin.md)
