<!--
 * @Date: 2020-09-28 20:48:01
 * @LastEditors: 曾令宇
 * @LastEditTime: 2020-09-30 18:47:50
 * @FilePath: \gx-demo\src\views\About.vue
-->
<template>
  <div id="container" class="about">
    <canvas id="map"></canvas>
    <canvas id="name"></canvas>
  </div>
</template>
<script>
import * as Three from "three";
import * as D3 from "d3-geo";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
// import { TrackballControls } from "three/examples/jsm/controls/TrackballControls";
export default {
  name: "about",
  data() {
    return {
      camera: undefined,
      scene: undefined,
      controls: undefined,
      renderer: undefined,
      map: undefined,
      raycaster: undefined,
      mouse: undefined,
      activeInstersect: [],
    };
  },
  methods: {
    loadGeoData() {
      const loader = new Three.FileLoader();
      const promise = new Promise((res, rej) => {
        loader.load(
          "/geo/guangxi.json",
          (data) => {
            res(JSON.parse(data));
          },
          () => {},
          (err) => {
            rej(err);
          }
        );
      });
      return promise;
    },
    initMap(geoJson) {
      const map = new Three.Object3D();
      const [cx, cy] = [108.320004, 22.82402];
      const projection = D3.geoMercator()
        .center([cx, cy])
        .scale(400)
        .translate([0, 0]);
      geoJson.features.forEach((el) => {
        const city = new Three.Object3D();
        console.log(city);
        const coordinates = el.geometry.coordinates;
        const extrudeSettings = {
          depth: 1,
          bevelEnabled: false,
        };
        coordinates.forEach((multiPolygon) => {
          multiPolygon.forEach((polygon) => {
            const shape = new Three.Shape();
            const lineMaterial = new Three.LineBasicMaterial({
              color: "white",
            });
            const lineGeometry = new Three.Geometry();
            polygon.forEach((grid, index) => {
              const [x, y] = projection(grid);
              if (index === 0) {
                shape.moveTo(x, -y);
              }
              shape.lineTo(x, -y);
              lineGeometry.vertices.push(new Three.Vector3(x, -y, 1.01));
            });
            const geometry = new Three.ExtrudeGeometry(shape, extrudeSettings);
            const material = new Three.MeshBasicMaterial({
              color: "#02A1E2",
              transparent: false,
              opacity: 1,
            });
            const material1 = new Three.MeshBasicMaterial({
              color: "#3480C4",
              transparent: false,
              opacity: 1,
            });
            const mesh = new Three.Mesh(geometry, [material, material1]);
            const line = new Three.Line(lineGeometry, lineMaterial);
            city.add(mesh);
            city.add(line);
          });
        });
        city.properties = el.properties;
        if (el.properties.centroid) {
          const [x, y] = projection(el.properties.centroid);
          city.properties._centroid = [x, y];
          if (city.properties.name) {
            this.createText(city.properties.name, x, -y, 1.01).then((t) => {
              city.add(t);
            });
          }
        }
        map.add(city);
      });
      const [xx, yy] = projection([cx, cy]);
      this.camera.position.set(xx, yy - 30, 35);
      this.camera.lookAt(xx, yy, 0);
      return map;
    },
    setLight() {
      let ambientLight = new Three.AmbientLight(0xffffff); // 环境光
      this.scene.add(ambientLight);
      this.controls.update();
    },
    setControls() {
      this.controls = new OrbitControls(
        this.camera,
        this.renderer.domElement
      );
      this.controls.enablePan = false;
    },
    setRaycaster() {
      this.raycaster = new Three.Raycaster();
      this.mouse = new Three.Vector2();
      const _this = this;
      const mouseMove = (event) => {
        _this.mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
        _this.mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
        _this.raycaster.setFromCamera(_this.mouse, _this.camera);
        console.log(event);
        console.log(window.innerWidth);
        console.log(window.innerHeight);
        console.log(_this.mouse);
        const intersects = _this.raycaster.intersectObjects(
          _this.scene.children,
          true
        );
        if (_this.activeInstersect && _this.activeInstersect.length > 0) {
          _this.activeInstersect.forEach((intersect) => {
            // intersect.object.parent.position.z = 0;
            intersect.object.parent.translateZ(-3);
          });
        }
        _this.activeInstersect = [];
        if (intersects && intersects.length > 0) {
          // console.log(intersects);
          // intersects.forEach((intersect) => {});
          for (const intersect of intersects) {
            if (
              intersect.object.material &&
              intersect.object.material.length > 0
            ) {
              intersect.object.parent.translateZ(3);
              _this.activeInstersect.push(intersect);
              // console.log(intersect);
              break;
            }
          }
        }
      };
      // window.addEventListener("mousemove", mouseMove, false);
      // window.addEventListener(
      //   "click",
      //   (event) => {
      //     console.log(event);
      //     console.log(window.innerWidth);
      //     console.log(window.innerHeight);
      //   },
      //   false
      // );

      window.addEventListener("mousemove", mouseMove, false);
    },
    async createText(text, x = 0, y = 0, z = 0) {
      return new Promise((res, rej) => {
        const fontLoader = new Three.FontLoader();
        let result;
        try {
          fontLoader.load("/Microsoft YaHei_Regular.json", (font) => {
            var fontMaterial = new Three.MeshLambertMaterial({
              color: 0x333333,
              side: Three.DoubleSide,
            });
            const shape = font.generateShapes(text, 0.4);
            const fontGeometry = new Three.ShapeGeometry(shape);
            fontGeometry.computeBoundingBox();
            result = new Three.Mesh(fontGeometry, fontMaterial);
            result.position.set(x, y, z);
            res(result);
          });
        } catch (e) {
          rej(e);
        }
      });
    },
    showName() {
      const width = window.innerWidth;
      const height = window.innerHeight;
      let canvas = document.querySelector("#name");
      if (!canvas) {
        return;
      }
      canvas.innerHTML = "";
      canvas.width = width;
      canvas.height = height;
      const ctx = canvas.getContext("2d");
      const offCanvas = document.createElement("canvas");
      offCanvas.width = width;
      offCanvas.height = height;
      const ctxOffCanvas = offCanvas.getContext("2d");
      ctxOffCanvas.font = "16.5px Arial";
      ctxOffCanvas.strokeStyle = "#FFFFFF";
      ctxOffCanvas.fillStyle = "#000000";
      const texts = [];
      this.map.children.forEach((el) => {
        if (!el.properties._centroid) {
          return;
        }
        const x = el.properties._centroid[0];
        const y = -el.properties._centroid[1];
        const z = 1.1;
        const vector = new Three.Vector3(x, y, z);
        // const position = vector.project(this.camera);
        const name = el.properties.name;
        const left = ((vector.x + 1) / 2) * width;
        const top = ((vector.y - 1) / 2) * height;
        const text = {
          name,
          left,
          top,
          width: ctxOffCanvas.measureText(name).width,
          height: 16.5,
        };
        let show = true;
        for (let t of texts) {
          if (
            text.left + text.width < t.left ||
            text.top + text.height < t.top ||
            t.left + t.width < text.left ||
            t.top + t.height < text.top
          ) {
            show = true;
          } else {
            show = false;
            break;
          }
        }
        if (show) {
          texts.push(text);
          ctxOffCanvas.strokeText(name, left, top);
          ctxOffCanvas.fillText(name, left, top);
        }
      });
      ctx.drawImage(offCanvas, 0, 0);
    },
    render() {
      this.renderer.render(this.scene, this.camera);
    },
    animate() {
      // this.showName();
      requestAnimationFrame(this.animate);
      this.render();
      this.controls.update();
    },
    setWindowSize() {
      this.containerEl.style.height = `${window.innerHeight}px`;
      this.containerEl.style.width = `${window.innerWidth}px`;
    },
    onResize() {
      this.setWindowSize();
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      this.render();
    },
  },
  mounted() {
    this.renderer = new Three.WebGLRenderer({
      alpha: true,
      canvas: document.querySelector("#map"),
    });
    this.renderer.setSize(window.innerWidth, window.innerHeight);
    this.containerEl.append(this.renderer.domElement);
    this.scene = new Three.Scene();
    this.camera = new Three.PerspectiveCamera(
      45,
      window.innerWidth / window.innerHeight,
      0.1,
      2000
    );
    this.setControls();
    this.setRaycaster();
    this.loadGeoData().then((data) => {
      this.scene.add((this.map = this.initMap(data)));
      this.animate();
    });
    window.addEventListener("resize", this.onResize);
  },
  computed: {
    clientWidth() {
      // return this.containerEl.clientWidth;
      return window.innerWidth;
    },
    clientHeight() {
      // return this.containerEl.clientHeight;
      return window.innerHeight;
    },

    containerEl() {
      return document.getElementById("container");
    },
  },
};
</script>
<style lang="less" scoped>
#container {
  width: 100%;
  position: absolute;
  top: 0;
  left: 0;
  // display: none;
  canvas {
    width: 100%;
    height: 100%;
  }
  #name {
    position: absolute;
    top: 0;
    left: 0;
    pointer-events: none;
  }
}
</style>
