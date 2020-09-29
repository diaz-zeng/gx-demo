<!--
 * @Date: 2020-09-28 20:48:01
 * @LastEditors: 曾令宇
 * @LastEditTime: 2020-09-30 00:06:51
 * @FilePath: \gx-demo\src\views\About.vue
-->
<template>
  <div id="container" class="about"></div>
</template>
<script>
import * as Three from "three";
import * as D3 from "d3-geo";
// import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { TrackballControls } from "three/examples/jsm/controls/TrackballControls";
export default {
  name: "about",
  data() {
    return {
      camera: undefined,
      scene: undefined,
      controls: undefined,
      renderer: undefined,
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
        .scale(80)
        .translate([0, 0]);
      geoJson.features.forEach((el) => {
        const city = new Three.Object3D();
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
              lineGeometry.vertices.push(new Three.Vector3(x, -y, 1));
            });
            const geometry = new Three.ExtrudeGeometry(shape, extrudeSettings);
            const material = new Three.MeshBasicMaterial({
              color: "#02A1E2",
              transparent: true,
              opacity: 0.6,
            });
            const material1 = new Three.MeshBasicMaterial({
              color: "#3480C4",
              transparent: true,
              opacity: 0.5,
            });
            const mesh = new Three.Mesh(geometry, [material, material1]);
            const line = new Three.Line(lineGeometry, lineMaterial);
            city.add(mesh);
            city.add(line);
          });
        });
        city.properties = el.properties;
        if (el.properties.contorid) {
          const [x, y] = projection(el.properties.contorid);
          city.properties._centroid = [x, y];
        }
        map.add(city);
      });
      // this.camera.position.set(map.position.x, map.position.y, 50);
      // this.camera.lookAt(map.position);
      const [xx, yy] = projection([cx, cy]);
      console.log([xx, yy]);
      this.camera.position.set(xx, -yy, 30);
      this.camera.lookAt(map.position);
      // this.camera.rotation.set(-90, 0, 0);
      // new Three.Camera().lookAt();
      // new Three.Camera().position.set()
      // this.camera.position.set(0, -70, 90);
      // this.camera.lookAt(0, 0, 0);
      return map;
    },
    setLight() {
      let ambientLight = new Three.AmbientLight(0xffffff); // 环境光
      this.scene.add(ambientLight);
      this.controls.update();
    },
    setControls() {
      this.controls = new TrackballControls(
        this.camera,
        this.renderer.domElement
      );
      this.controls.enablePan = false;
    },
    // setViewer(x,y,z){
    //   this.camera.lookAt()
    // },
    render() {
      this.renderer.render(this.scene, this.camera);
    },
    animate() {
      requestAnimationFrame(this.animate);
      this.controls.update();
      this.render();
      // console.log(this.camera);
      this.controls.update();
    },
  },
  mounted() {
    this.renderer = new Three.WebGLRenderer();
    // this.renderer.setClearColor(0xb9d3ff, 1);
    this.renderer.setSize(this.clientWidth, this.clientHeight);
    this.containerEl.append(this.renderer.domElement);
    // document.body.appendChild(this.renderer.domElement);
    this.scene = new Three.Scene();
    this.camera = new Three.PerspectiveCamera(
      45,
      this.clientWidth / this.clientHeight,
      0.1,
      1000
    );
    // this.camera.position.set(this.clientWidth / 2, this.clientHeight / 2, 150);
    // this.camera.lookAt(0, 0, 0);
    this.setControls();
    // this.setLight();
    this.loadGeoData().then((data) => {
      this.scene.add(this.initMap(data));
      this.animate();
    });
  },
  computed: {
    clientWidth() {
      return this.containerEl.clientWidth;
      // return window.innerWidth;
    },
    clientHeight() {
      return this.containerEl.clientHeight;
      // return window.innerHeight;
    },

    containerEl() {
      return document.getElementById("container");
    },
  },
};
</script>
<style lang="less" scoped>
#container {
  min-width: 1366px;
  min-height: 768px;
  // width: 100%;
  // height: 100%;
  // display: none;
}
</style>
