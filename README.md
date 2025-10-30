# Sistema Solar Interactivo 3D con Three.js
### Simulación tridimensional del sistema solar con cámaras interactivas, eclipses y modo de vuelo

---

**Proyecto académico desarrollado por Jesús Alberto Ortega Hernández**  
*Aplicación educativa que combina gráficos 3D, animación orbital y control de cámara en tiempo real.*

---

## Descripción general

El **Sistema Solar Interactivo 3D** es una simulación creada con **Three.js** que representa los planetas orbitando alrededor del Sol con texturas realistas, efectos de luz y sombra, y una interfaz interactiva para cambiar el modo de cámara o seguir un planeta específico.  

Incluye tres modos de cámara, una simulación de eclipses entre Sol–Tierra–Luna y un sistema de movimiento en primera persona con una nave espacial.

---

## Vista previa y demostraciones

**Vídeo de demostración:** [https://www.youtube.com/watch?v=6H7yFnXkxbg](https://www.youtube.com/watch?v=6H7yFnXkxbg)  
**Versión online (CodeSandbox):** [https://codesandbox.io/p/sandbox/ig2526-s7-forked-l277mt](https://codesandbox.io/p/sandbox/ig2526-s7-forked-l277mt)

---
## Imágenes del proyecto:
---


## Planetas:

<img width="1607" height="915" alt="Captura de pantalla 2025-10-30 204151" src="https://github.com/user-attachments/assets/bd6247e2-3539-4517-8296-215b73271a7a" />


## Vista en modo planetas:

<img width="1635" height="884" alt="Captura de pantalla 2025-10-30 204650" src="https://github.com/user-attachments/assets/dc0a4d71-94e9-4552-9f9c-7aa2c644f969" />


## Vista en modo nave:

<img width="1636" height="904" alt="Captura de pantalla 2025-10-30 204807" src="https://github.com/user-attachments/assets/dd6a1f6c-2a86-4f45-aaa6-8e68d5c9016e" />


## Tareas realizadas

| Tarea | Descripción |
|-------------|-------------|
| Generar al menos cinco planetas | Consiste en definir un sistema solar conformado por cinco o mas planetas |
| Añadir orbita | Genera una orbita al rededor del sol para cada planeta |
| Añadir sombras | La luminosidad viene del sol, por lo que las caras que no miran a este tienen sombra |
| Añadir segunda textura | La Tierra, Saturno y Neptuno tienen una doble capa de textura para las nubes y los anillos |
| Generar eclipse | La luna genera un eclipse sobre la tierra |
| dat.GUI | Interfaz para modificar parámetros en tiempo real. |
| Modos de vista | Añadido modo de vista para centrarse en cada planeta y modo nave |


## Tecnologías utilizadas

| Tecnología | Descripción |
|-------------|-------------|
| Three.js | Motor 3D basado en WebGL. |
| dat.GUI | Interfaz para modificar parámetros en tiempo real. |
| OrbitControls | Control orbital de cámara mediante ratón. |
| JavaScript (ES6+) | Lógica principal y animaciones. |
| HTML / CSS | Estructura y estilos base. |
| Parcel | Empaquetador y servidor local. |

---

## Controles

| Entrada | Acción |
|----------|--------|
| W / S | Avanzar / Retroceder |
| A / D | Desplazarse lateralmente |
| Q / E | Ascender / Descender |
| Ratón | Rotar cámara (modo Normal) |
| dat.GUI | Cambiar modo de cámara o planeta a seguir |

---

## Modos de cámara

1. **Normal:** Control libre con el ratón usando OrbitControls.  
2. **Orbital:** La cámara sigue automáticamente el planeta seleccionado.  
3. **Nave:** Vista en primera persona con un modelo 3D de nave que sigue la orientación de la cámara.

---

## Estructura del proyecto

```
src/
├── index.html
├── main.js
├── space.jpg
├── sun.jpg
├── earthmap1k.jpg
├── earthcloudmap.jpg
├── earthcloudmaptrans_invert.jpg
├── moon.jpg
├── mercury.jpg
├── venus.jpg
├── mars.jpg
├── jupiter.jpg
├── saturno.jpg
├── saturn_rings.png
├── urano.jpg
└── neptuno.jpg
```

---

## Requisitos y ejecución

- Node.js v16 o superior  
- NPM v8 o superior  
- Navegador compatible con WebGL2

---

## Arquitectura general

- Renderizado: motor WebGL con Three.js.  
- Iluminación: PointLight (Sol) y AmbientLight (suaviza sombras).  
- Controles: cámara orbital y modo vuelo.  
- Interfaz: dat.GUI para cambiar modos y planetas.  
- Eclipses: cálculo mediante producto escalar Sol–Tierra–Luna.  
- Animación: requestAnimationFrame() actualiza posiciones y rotaciones.  

---

## Funciones explicadas en detalle

### Variables globales

```js
let scene, camera, renderer, controls;
let estrella;
let Planetas = [];
let t0 = 0;
let accglobal = 0.001;
let timestamp;

let keys = {};
let velocity = new THREE.Vector3();
let direction = new THREE.Vector3();
let moveSpeed = 0.2;

let modoCamara = "Normal";
let gui;
let planetaEnfocado = null;
let anguloCamara = 0;
let velocidadOrbitaCamara = 0.003;

let luna, sombraLuna, luzSolar, nave;
```

**Descripción:**
Estas variables definen los elementos principales de la simulación:

- `scene`: contenedor principal donde se agregan todos los objetos (planetas, luces, cámara).  
- `camera`: define la vista del usuario mediante proyección en perspectiva.  
- `renderer`: renderiza la escena en el canvas HTML usando WebGL.  
- `controls`: instancia de OrbitControls para controlar cámara con ratón.  
- `estrella`: malla que representa el Sol.  
- `Planetas`: arreglo con todos los planetas para iterarlos en animación.  
- `t0`: tiempo inicial de simulación.  
- `accglobal`: escala del tiempo (acelera o ralentiza movimiento).  
- `velocity` y `direction`: vectores para calcular movimiento de cámara en modo nave.  
- `moveSpeed`: velocidad de traslación de la cámara.  
- `modoCamara`: modo actual (“Normal”, “Orbital” o “Nave”).  
- `planetaEnfocado`: referencia al planeta que sigue la cámara en modo orbital.  
- `luzSolar`: luz principal que emula el Sol.  
- `nave`: grupo de objetos que forma el modelo de la nave espacial.  

---

### init()

```js
function init() : void
```

**Explicación:**  
Función principal de inicialización.  
Se encarga de configurar todos los componentes necesarios para que la simulación funcione correctamente.

**Elementos usados:**
- THREE.Scene(): crea el espacio 3D donde se añadirán los objetos.  
- THREE.PerspectiveCamera(): simula la percepción humana (ángulo de visión, distancia).  
- THREE.WebGLRenderer(): renderiza los gráficos usando la GPU.  
- THREE.PointLight(): luz focal que representa la iluminación del Sol.  
- THREE.AmbientLight(): luz ambiental para suavizar contrastes y sombras.  
- THREE.TextureLoader(): carga imágenes usadas como texturas en los planetas.  
- THREE.SphereGeometry(): se usa para crear el cielo invertido (entorno espacial).  
- OrbitControls: controla la cámara con el ratón.  

**Pasos principales:**
1. Crea la escena, la cámara y el renderizador.  
2. Configura el tamaño del render y habilita sombras (renderer.shadowMap.enabled = true).  
3. Añade luces (luzSolar, luzAmbiente).  
4. Crea el fondo estelar con una esfera invertida (BackSide).  
5. Llama a Estrella() para crear el Sol.  
6. Crea planetas con diferentes parámetros (Planeta()).  
7. Genera Luna y nave (crearLuna(), crearNave()).  
8. Configura el panel de interfaz (dat.GUI) con modos y planeta a seguir.  
9. Escucha eventos de teclado (keydown, keyup) y redimensionado (resize).  

---

### Estrella(rad, texturePath)

```js
function Estrella(rad, texturePath)
```

**Uso de elementos:**
- THREE.SphereGeometry(rad, 64, 64): genera la geometría esférica del Sol.  
- THREE.TextureLoader().load(texturePath): carga la textura del Sol.  
- THREE.MeshBasicMaterial: material sin sombreado, ideal para objetos autoiluminados.  
- scene.add(estrella): agrega el Sol a la escena.  

**Por qué se usa:**  
El Sol no necesita sombras ni reflejos, por eso se usa MeshBasicMaterial.  
La luz proviene del PointLight central (luzSolar), no del propio material.

---

### Planeta(nombre, radio, dist, vel, col, f1, f2, texturePath)

```js
function Planeta(nombre, radio, dist, vel, col, f1, f2, texturePath)
```

**Uso de elementos:**
- THREE.SphereGeometry(radio, 32, 32): define la forma esférica del planeta.  
- THREE.MeshPhongMaterial: material que reacciona a la luz, ideal para simular brillo y sombras.  
- THREE.EllipseCurve(): genera la trayectoria orbital visible (una elipse).  
- THREE.LineLoop(): dibuja la línea de órbita en el plano XZ.  
- THREE.RingGeometry(): crea los anillos de Saturno y Urano.  
- THREE.TextureLoader(): carga las texturas correspondientes.  

**Por qué se usa cada elemento:**  
- MeshPhongMaterial permite reflejos suaves según la iluminación, dando apariencia realista.  
- EllipseCurve y LineLoop muestran la órbita visualmente al usuario.  
- RingGeometry + DoubleSide + transparent: true se usa para crear anillos con canal alfa visible desde ambos lados.

**Casos especiales:**
- Tierra: añade nubes con textura semitransparente (uso de alphaMap y transparent).  
- Saturno y Urano: incluyen anillos con diferente inclinación (rotation.x).

---

### crearLuna()

```js
function crearLuna()
```

**Uso de elementos:**
- THREE.SphereGeometry(0.27, 32, 32): crea la geometría de la Luna.  
- THREE.MeshPhongMaterial: material que interactúa con la luz solar.  
- scene.add(luna): añade la Luna al sistema.  

**Por qué se usa:**  
MeshPhongMaterial permite reflejar luz del Sol sobre la Luna, generando un brillo realista.  
Los datos userData permiten definir su órbita y velocidad, actualizados cada frame.

---

### crearNave()

```js
function crearNave()
```

**Uso de elementos:**
- THREE.ConeGeometry(): forma principal del cuerpo de la nave.  
- THREE.BoxGeometry(): alas laterales.  
- THREE.Group(): agrupa las piezas de la nave para moverlas juntas.  

**Por qué se usa:**  
Group facilita mover todos los componentes a la vez (alas y cuerpo).  
Se oculta (visible = false) hasta activar el modo “Nave”.

---

### cambiarModoCamara(modo)

```js
function cambiarModoCamara(modo)
```

**Uso de elementos:**
- controls.enabled: habilita o deshabilita OrbitControls.  
- camera.position.set(): reposiciona la cámara según el modo.  
- nave.visible: muestra u oculta la nave.  

**Por qué se usa:**  
Permite alternar entre control libre (Normal), seguimiento automático (Orbital) o inmersivo (Nave).

---

### enfocarPlaneta(nombre)

```js
function enfocarPlaneta(nombre)
```

**Uso de elementos:**
- Array.find(): busca el planeta en el array Planetas.  
- toLowerCase(): permite búsqueda sin distinción de mayúsculas.  

**Por qué se usa:**  
Permite seleccionar el planeta objetivo del modo “Orbital” de forma dinámica.

---

### updateCameraMovement()

```js
function updateCameraMovement()
```

**Uso de elementos:**
- THREE.Vector3(): define direcciones tridimensionales de movimiento.  
- camera.getWorldDirection(): obtiene la dirección hacia la que apunta la cámara.  
- camera.position.add(): mueve la cámara en el espacio según la dirección calculada.  

**Por qué se usa:**  
Permite mover la cámara en modo vuelo con teclas.  
La nave sigue la cámara con un pequeño offset, creando sensación de desplazamiento real.

---

### onWindowResize()

```js
function onWindowResize()
```

**Uso de elementos:**
- camera.aspect: ajusta la proporción de pantalla.  
- camera.updateProjectionMatrix(): actualiza la proyección de la cámara.  
- renderer.setSize(): ajusta el tamaño del canvas al nuevo tamaño de la ventana.  

**Por qué se usa:**  
Mantiene la proporción correcta del render al redimensionar la ventana.

---

### animationLoop()

```js
function animationLoop()
```

**Uso de elementos:**
- requestAnimationFrame(animationLoop): bucle continuo de actualización.  
- Math.cos() / Math.sin(): calculan la posición orbital de cada planeta.  
- renderer.render(scene, camera): dibuja la escena.  
- Vector3.dot(): calcula el producto escalar usado en detección de eclipses.  

**Por qué se usa:**  
Permite mantener movimiento fluido y continuo.  
requestAnimationFrame optimiza el render para sincronizarse con los FPS del monitor.

---

## Cálculo de eclipses

```js
const vectorSolTierra = new THREE.Vector3()
  .subVectors(tierra.position, estrella.position)
  .normalize();

const vectorTierraLuna = new THREE.Vector3()
  .subVectors(luna.position, tierra.position)
  .normalize();

const alineacion = vectorSolTierra.dot(vectorTierraLuna);

if (alineacion < -0.995) {
  // Eclipse activo
}
```

**Explicación técnica:**
- subVectors(a, b): crea un vector dirección desde b hasta a.  
- normalize(): convierte el vector a una longitud unitaria (magnitud 1).  
- dot(): devuelve el producto escalar entre ambos vectores.  
- Si el resultado es cercano a -1, están alineados en direcciones opuestas (eclipse).  

---

## Rendimiento y ajustes

- Reducir shadow.mapSize mejora rendimiento (a costa de calidad de sombra).  
- Cambiar accglobal ajusta la velocidad global de movimiento.  
- Reducir segmentos en geometrías (SphereGeometry) aumenta FPS.  
- Texturas más pequeñas reducen uso de VRAM.

---

## Extensiones recomendadas

```js
// Añadir nuevo planeta
Planeta("Plutón", 0.3, 32.0, 0.12, 0xcccccc, 1.0, 0.95, "src/pluton.jpg");
```

```js
// Añadir anillos personalizados
const ringTex = loader.load("src/saturn_rings.png");
const ringGeo = new THREE.RingGeometry(radio * 1.3, radio * 2.4, 128);
const ringMat = new THREE.MeshPhongMaterial({
  map: ringTex,
  transparent: true,
  side: THREE.DoubleSide,
  opacity: 0.75
});
const anillos = new THREE.Mesh(ringGeo, ringMat);
anillos.rotation.x = Math.PI / 3;
planeta.add(anillos);
```

---


---

## Créditos de texturas

- Tierra y nubes: NASA Visible Earth.  
- Anillos: texturas libres con canal alfa.  
- Fondo estelar: textura bajo licencia Creative Commons.  

---

## Licencia y autoría

**Autor:** Jesús Alberto Ortega Hernández  
**Proyecto:** Simulación Interactiva 3D (Ingeniería Informática)  
**Año:** 2025  

---
