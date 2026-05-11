# Canvas, карты, 3D — Junior

## Вопросы

- [Что такое HTML5 Canvas и для чего он нужен?](#что-такое-html5-canvas-и-для-чего-он-нужен)
- [Как рисовать на Canvas?](#как-рисовать-на-canvas)
- [Как анимировать Canvas с requestAnimationFrame?](#как-анимировать-canvas-с-requestanimationframe)
- [Как добавить интерактивную карту через Leaflet?](#как-добавить-интерактивную-карту-через-leaflet)
- [Что такое Three.js и как работает базовая сцена?](#что-такое-threejs-и-как-работает-базовая-сцена)

---

## Что такое HTML5 Canvas и для чего он нужен?

`<canvas>` — HTML-элемент для растровой 2D/3D-графики через JavaScript API. Рисует попиксельно, не имеет DOM-элементов внутри (в отличие от SVG).

**Когда использовать:**
- Динамические графики и диаграммы (Chart.js)
- Игры (спрайты, физика, collision detection)
- Обработка изображений (фильтры, кроп)
- Визуализация данных с тысячами объектов
- Рендеринг через WebGL (Three.js)

**Когда НЕ использовать:**
- Статическая иконографика → SVG (масштабируется, доступен)
- Интерактивные диаграммы с tooltip → SVG/D3.js (DOM события)
- Обычные UI компоненты → HTML/CSS

**Основные отличия от SVG:**
| | Canvas | SVG |
|---|---|---|
| Рендеринг | Растровый (пиксели) | Векторный (масштабируемый) |
| DOM | Нет (один элемент) | Есть (каждый объект — элемент) |
| Производительность | Лучше для 10 000+ объектов | Лучше для 100–1000 |
| Доступность | Сложно | Встроена |
| Hit-testing | Вручную | Автоматически |

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как рисовать на Canvas?

Получить контекст → использовать 2D API.

```typescript
const canvas = document.getElementById('myCanvas') as HTMLCanvasElement;
const ctx = canvas.getContext('2d')!;

// Размеры (важно устанавливать через атрибут, не CSS — иначе размытие):
canvas.width = 800;
canvas.height = 600;

// === Прямоугольники ===
ctx.fillStyle = '#3b82f6';     // цвет заливки
ctx.fillRect(10, 10, 100, 50); // x, y, width, height

ctx.strokeStyle = '#1e40af';   // цвет обводки
ctx.lineWidth = 2;
ctx.strokeRect(10, 70, 100, 50);

ctx.clearRect(0, 0, canvas.width, canvas.height); // очистить область

// === Пути (paths) ===
ctx.beginPath();
ctx.moveTo(50, 50);       // переместить перо без рисования
ctx.lineTo(200, 50);      // линия
ctx.lineTo(125, 150);     // ещё линия
ctx.closePath();           // закрыть путь (соединить с начальной точкой)
ctx.fillStyle = 'tomato';
ctx.fill();
ctx.stroke();

// === Окружности ===
ctx.beginPath();
ctx.arc(
  200, 200, // центр x, y
  50,       // радиус
  0,        // начальный угол (радианы)
  Math.PI * 2 // конечный угол (2π = полный круг)
);
ctx.fillStyle = 'lime';
ctx.fill();

// === Текст ===
ctx.font = '24px Arial';
ctx.fillStyle = 'black';
ctx.fillText('Hello Canvas', 50, 300);
ctx.strokeText('Outline', 50, 340);

// === Изображения ===
const img = new Image();
img.onload = () => {
  ctx.drawImage(img, 0, 0, 200, 150); // x, y, width, height
};
img.src = '/photo.jpg';

// === Трансформации ===
ctx.save();              // сохранить состояние матрицы
ctx.translate(100, 100); // переместить начало координат
ctx.rotate(Math.PI / 4); // повернуть на 45°
ctx.scale(2, 2);          // масштаб
ctx.fillRect(0, 0, 50, 50);
ctx.restore();           // восстановить состояние
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как анимировать Canvas с requestAnimationFrame?

`requestAnimationFrame` — браузерный API для анимаций. Вызывается перед следующим перерисованием экрана (~60 fps). Лучше `setInterval` — автоматически паузится вне вкладки, синхронизируется с refresh rate.

```typescript
// Базовый паттерн анимации:
let animationId: number;

const draw = (timestamp: number) => {
  // 1. Очистить canvas
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // 2. Обновить состояние
  x = (x + speed) % canvas.width;

  // 3. Нарисовать
  ctx.beginPath();
  ctx.arc(x, 100, 20, 0, Math.PI * 2);
  ctx.fillStyle = '#3b82f6';
  ctx.fill();

  // 4. Запросить следующий кадр
  animationId = requestAnimationFrame(draw);
};

// Запуск:
animationId = requestAnimationFrame(draw);

// Остановка (важно при unmount компонента!):
cancelAnimationFrame(animationId);
```

**Интеграция в React:**
```tsx
const CanvasAnimation = () => {
  const canvasRef = useRef<HTMLCanvasElement>(null);
  const animRef = useRef<number>(0);

  useEffect(() => {
    const canvas = canvasRef.current!;
    const ctx = canvas.getContext('2d')!;
    let x = 0;

    const draw = () => {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.fillRect(x, 50, 30, 30);
      x = (x + 2) % canvas.width;
      animRef.current = requestAnimationFrame(draw);
    };

    animRef.current = requestAnimationFrame(draw);

    return () => cancelAnimationFrame(animRef.current); // cleanup!
  }, []);

  return <canvas ref={canvasRef} width={600} height={200} />;
};
```

**Оптимизация производительности:**
- `ctx.clearRect` только нужной области, а не всего canvas
- Off-screen canvas — рисовать в скрытый canvas, копировать в видимый
- Минимизировать state changes (`fillStyle`, `font`) — они дорогостоящие
- Избегать `getImageData/putImageData` в каждом кадре — очень медленно

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Как добавить интерактивную карту через Leaflet?

**Leaflet** — лёгкая open-source библиотека интерактивных карт.

```bash
npm install leaflet
npm install --save-dev @types/leaflet
```

**Базовая интеграция в React:**
```tsx
import { useEffect, useRef } from 'react';
import L from 'leaflet';
import 'leaflet/dist/leaflet.css'; // важно!

// Исправление иконок по умолчанию в webpack/vite:
import markerIconUrl from 'leaflet/dist/images/marker-icon.png';
import markerIcon2xUrl from 'leaflet/dist/images/marker-icon-2x.png';
import markerShadowUrl from 'leaflet/dist/images/marker-shadow.png';

delete (L.Icon.Default.prototype as any)._getIconUrl;
L.Icon.Default.mergeOptions({
  iconUrl: markerIconUrl,
  iconRetinaUrl: markerIcon2xUrl,
  shadowUrl: markerShadowUrl,
});

const MapComponent = () => {
  const containerRef = useRef<HTMLDivElement>(null);
  const mapRef = useRef<L.Map | null>(null);

  useEffect(() => {
    if (mapRef.current) return; // защита от двойного mount (React StrictMode)

    mapRef.current = L.map(containerRef.current!, {
      center: [55.7558, 37.6173], // Москва
      zoom: 13,
    });

    // Тайловый слой (карта):
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '© OpenStreetMap contributors',
    }).addTo(mapRef.current);

    // Маркер:
    const marker = L.marker([55.7558, 37.6173])
      .addTo(mapRef.current)
      .bindPopup('<b>Центр Москвы</b>');

    // Кружок:
    L.circle([55.75, 37.61], { radius: 500, color: 'blue', fillOpacity: 0.2 })
      .addTo(mapRef.current);

    return () => {
      mapRef.current?.remove(); // cleanup!
      mapRef.current = null;
    };
  }, []);

  return <div ref={containerRef} style={{ height: '400px', width: '100%' }} />;
};
```

**Кластеризация маркеров (Leaflet.markercluster):**
```typescript
import 'leaflet.markercluster';
import 'leaflet.markercluster/dist/MarkerCluster.css';

const markers = L.markerClusterGroup();
points.forEach(({ lat, lng, label }) => {
  markers.addLayer(L.marker([lat, lng]).bindPopup(label));
});
mapRef.current!.addLayer(markers);
```

**Получение координат по клику:**
```typescript
mapRef.current!.on('click', (e: L.LeafletMouseEvent) => {
  console.log('Координаты:', e.latlng.lat, e.latlng.lng);
});
```

**Геолокация браузера:**
```typescript
navigator.geolocation.getCurrentPosition(
  ({ coords }) => {
    const { latitude, longitude } = coords;
    mapRef.current!.setView([latitude, longitude], 15);
    L.marker([latitude, longitude]).addTo(mapRef.current!).bindPopup('Вы здесь');
  },
  (error) => console.error('Геолокация недоступна:', error),
  { enableHighAccuracy: true, timeout: 5000 }
);
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->

---

## Что такое Three.js и как работает базовая сцена?

**Three.js** — библиотека 3D-графики поверх WebGL. Абстрагирует низкоуровневый WebGL API до понятных объектов.

**Основные концепции:**
- **Scene** — контейнер для всех объектов
- **Camera** — точка наблюдения (PerspectiveCamera / OrthographicCamera)
- **Renderer** — рисует сцену на canvas через WebGL
- **Mesh** = Geometry (форма) + Material (внешний вид)
- **Light** — источник освещения

```bash
npm install three
npm install --save-dev @types/three
```

**Базовая сцена в React:**
```tsx
import { useEffect, useRef } from 'react';
import * as THREE from 'three';

const ThreeScene = () => {
  const mountRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const mount = mountRef.current!;
    const width = mount.clientWidth;
    const height = mount.clientHeight;

    // 1. Сцена:
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x1a1a2e);

    // 2. Камера:
    const camera = new THREE.PerspectiveCamera(
      75,           // Field of View (угол обзора в градусах)
      width / height, // соотношение сторон
      0.1,          // near clipping plane
      1000          // far clipping plane
    );
    camera.position.z = 5;

    // 3. Renderer:
    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(width, height);
    renderer.setPixelRatio(window.devicePixelRatio);
    mount.appendChild(renderer.domElement);

    // 4. Объект (куб):
    const geometry = new THREE.BoxGeometry(1, 1, 1);
    const material = new THREE.MeshStandardMaterial({ color: 0x3b82f6 });
    const cube = new THREE.Mesh(geometry, material);
    scene.add(cube);

    // 5. Освещение:
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
    scene.add(ambientLight);

    const pointLight = new THREE.PointLight(0xffffff, 1);
    pointLight.position.set(5, 5, 5);
    scene.add(pointLight);

    // 6. Анимационный цикл:
    let animId: number;
    const animate = () => {
      animId = requestAnimationFrame(animate);
      cube.rotation.x += 0.01;
      cube.rotation.y += 0.01;
      renderer.render(scene, camera);
    };
    animate();

    // 7. Cleanup (важно!):
    return () => {
      cancelAnimationFrame(animId);
      renderer.dispose();
      geometry.dispose();
      material.dispose();
      mount.removeChild(renderer.domElement);
    };
  }, []);

  return <div ref={mountRef} style={{ width: '100%', height: '400px' }} />;
};
```

**Загрузка 3D-модели (GLB/GLTF):**
```typescript
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';

const loader = new GLTFLoader();
loader.load(
  '/model.glb',
  (gltf) => {
    scene.add(gltf.scene);
    // Запуск анимаций модели:
    const mixer = new THREE.AnimationMixer(gltf.scene);
    gltf.animations.forEach((clip) => mixer.clipAction(clip).play());
  },
  (progress) => console.log('Загрузка:', (progress.loaded / progress.total * 100) + '%'),
  (error) => console.error('Ошибка загрузки:', error)
);
```

**Орбитальные контролы (вращение мышью):**
```typescript
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';

const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true; // плавное торможение
controls.dampingFactor = 0.05;

// В animate():
controls.update(); // важно вызывать в каждом кадре
```

**React Three Fiber** — декларативный подход (аналог React для Three.js):
```tsx
import { Canvas } from '@react-three/fiber';
import { OrbitControls } from '@react-three/drei';

const App = () => (
  <Canvas camera={{ position: [0, 0, 5] }}>
    <ambientLight intensity={0.5} />
    <pointLight position={[5, 5, 5]} />
    <mesh rotation={[0.01, 0.01, 0]}>
      <boxGeometry args={[1, 1, 1]} />
      <meshStandardMaterial color="#3b82f6" />
    </mesh>
    <OrbitControls />
  </Canvas>
);
```

**Связанные задачи:**
<!-- Связанных задач нет -->

**Материалы для изучения:**
<!-- Материалы не добавлены -->
