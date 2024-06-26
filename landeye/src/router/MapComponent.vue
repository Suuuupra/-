<script>

import {Tile as TileLayer, Vector as VectorLayer} from "ol/layer";
import {Vector as VectorSource} from "ol/source";
import XYZ from 'ol/source/XYZ'
import {Map, View} from 'ol'
import {fromLonLat} from 'ol/proj'
import 'ol/ol.css'
import {defaults as defaultControls, FullScreen, ScaleLine} from "ol/control";
import Feature from 'ol/Feature';
import {Icon, Style, Fill, Stroke, Text} from "ol/style";
import GeoJSON from "ol/format/GeoJSON"
import landdata_geo from "../assets/landdata_geo.json";
import {Point, Polygon, LinearRing, LineString} from "ol/geom";
import {transform} from 'ol/proj';
import {onMounted, ref, toRefs, watch, nextTick, getCurrentInstance, inject, onUnmounted, computed} from "vue";
import {getDistance} from 'ol/sphere'
import axios from "axios";
import { inAndOut } from 'ol/easing'
export default {
  name: 'MapComponent',
  props: {
    msg: String,
    center: [],
  },
  setup(props, context) {
    let {msg: receive} = toRefs(props);
    var socket_data = ref('');
    var curve = ref('');
    var lonlat = ref('');
    const center = props.center
    watch(receive, watchHandle);

    const Points = [
      [118.759167,24.772778],
      [118.699444,24.773056],
      [118.718611,24.771944],
      [118.731111,24.756389],
      [118.730556,24.773056]
    ]
    let map
    let source
    let message = ref('')

    const mapUrl = 'http://120.37.123.14:58090/tiles/{z}/{x}/{y}.png';
    let cameraVectorLayerSource = null;
    let cameraVectorLayer = null;
    let pointVectorLayerSource = null;
    let pointVectorLayer = null;
    let WarnPoint = [];
    const coordinate = inject('coordinate');
    const cameraId = inject('cameraId');
    const coordinates = inject('coordinates');
    var warnPointVectorSource = new VectorSource(); // 创建一个空的矢量源用于警告点
    var warnPointVectorLayer = new VectorLayer({ // 创建矢量图层使用上述矢量源
      source: warnPointVectorSource,
      style: new Style({
        image: new Icon({
          anchor: [0.5, 1],
          scale: 1,
          src: '/img/warning.png' // 图标路径，根据需要调整
        })
      })
    });
    var CenterVectorSource = new VectorSource(); // 创建一个空的矢量源用于警告中心点
    var CenterVectorLayer = new VectorLayer({ // 创建矢量图层使用上述矢量源
      source: CenterVectorSource,
      style: new Style({
        image: new Icon({
          anchor: [0.5, 1],
          scale: 0.3,
          src: '/img/center.png'
        })
      })
    });
    var CameraVectorSource = new VectorSource();
    var CameraVectorLayer = new VectorLayer({
      source: CameraVectorSource,
      style: new Style({
        image: new Icon({
          anchor: [0.5, 1],
          scale: 1,
          src: '/img/camera.png'
        })
      })
    });
    let defaultCoordinate = [];

    onMounted(() => {
      console.log('cameraId',cameraId)
      console.log('cameraCoordiantes',coordinates)
      initMap();
      updateCameraFeature(Points);
      fetchData();
      nextTick(() => {
        source = new VectorSource({
          features: new GeoJSON().readFeatures(landdata_geo)
        });
        setInterval(() => {
          //drawFanFeature();
        }, 1000);
        setInterval(() => {
          //drawPoint();
        }, 1000);
        map.addEventListener('click', getCoordinates);
      });
    });

    // 监听 coordinate 的变化
    watch(coordinate, (newValue) => {
      console.log('Coordinate updated:', newValue);
      updateMapCenter(newValue);
      updateCenterFeature(newValue);
    }, {immediate: true});

    watch(coordinates,(newValue) =>{
      defaultCoordinate = [newValue[0],newValue[1]];
    },{immediate: true});
    function watchHandle() {
      socket_data.value = receive.value;
      var obj = JSON.parse(socket_data.value);
      if (obj.angle !== undefined) {
        curve.value = obj;
      } else if (obj.errorcode === 406 || obj.errorcode === 407) {
        curve.value = obj;
      } else if (obj.packetscr == 4) {
        lonlat.value = obj;
      } else if (obj.clearPoint == 1) {
        lonlat.value = obj;
      }
    }

    function sendMessage() {
      context.emit('change', message);
    }

    function initMap() {
      //const initialCenter = coordinate.length ? fromLonLat([coordinate[0], coordinate[1]]) : fromLonLat(defaultCoordinate);
      //warnPointVectorSource.clear();
      var Layers = [
        new TileLayer({
          title: "影像底图",
          source: new XYZ({
            url: mapUrl
          }),
        }),
        new VectorLayer({
          source: new VectorSource({
            url: '/landdata_geo.json',
            format: new GeoJSON()
          }),
          style: new Style({
            fill: new Fill({
              color: 'rgba(5, 95, 23, 0.4)'
            }),
            stroke: new Stroke({
              color: 'rgba(1, 220, 4)',
              width: 2
            })
          })
        }),//地块GeoJson
        warnPointVectorLayer,// 添加警告点图层
        CenterVectorLayer,//添加中心点图层
        CameraVectorLayer //添加摄像头点位图层
      ]
      map = new Map({
        layers: Layers,
        view: new View({
          center: fromLonLat(center),
          zoom: 16,
          minZoom: 8,
          maxZoom: 18
        }),
        target: 'map',
        controls: defaultControls().extend([
          new FullScreen(),
          new ScaleLine({
            units: 'metric'
          })
        ])
      })
      //drawFanFeature(curve);
      //drawPoint(lonlat);
    }

    function updateMapCenter(newCenter) {
      if (map && newCenter) {
        console.log('newcenter',newCenter)
        //map.getView().setCenter(fromLonLat(newCenter));
        const longitude = newCenter[0];
        const latitude = newCenter[1];
// 将经纬度转换为地图视图所使用的投影坐标
        const center = fromLonLat([longitude, latitude])
        const currentZoom = map.getView().getZoom();
// 缩小地图到某个级别（比如缩小到当前级别的一半）
     map.getView().animate({
          zoom: currentZoom - 1,
          duration: 800,
          easing: inAndOut
        }, function() {
          // 第一个动画完成后，执行移动到目标点的动画
          map.getView().animate({
            center: center,
            duration: 500,
            easing: inAndOut
          }, function() {
            // 移动到目标点后，执行放大动画
            map.getView().animate({
              zoom: 16, // 恢复到原来的缩放级别
              center: center, // 保持中心不变
              duration: 800,
              easing: inAndOut
            });
          });
        });
      }
    }
    function getCoordinates(event) {
      let lonlat = transform(event.coordinate, 'EPSG:3857', 'EPSG:4326');
      let lon = lonlat[0];
      let lat = lonlat[1];

      console.log([lon, lat])
      let data = {
        "packettype": 6,
        "cameraid": 320000003,
        "timestamp": Date.now(),
        "zoomtype": 0,
        "x": lon,
        "y": lat,
        "z": 1
      }
      message = JSON.stringify(data);
      sendMessage();
    }

    function drawFanFeature() {
      const lineStyle = new Style({
        stroke: new Stroke({
          color: 'rgba(255, 0, 0, 0.5)',
          width: 2,
        })
      });

      const fanStyle = new Style({
        fill: new Fill({
          color: 'rgba(32, 157, 230, 0.3)'
        }),
        stroke: new Stroke({
          color: 'rgba(255, 205, 67)',
          width: 2
        }),
        zIndex: 10000,
        anchor: [0.5, 1]
      });

      if (curve != "") {
        var params = curve
        let angle = OrientToAngle(params.bearing)
        const center = fromLonLat([120.29340255, 29.3125972])
        let lineCoordinates = []
        if (lonlat != "" && lonlat.coords !== undefined && lonlat.coords[0].x !== 0 && lonlat.coords[0].y !== 0) {
          let targetPoint = fromLonLat([lonlat.coords[0].x, lonlat.coords[0].y])
          lineCoordinates = [
            center,
            [targetPoint[0], targetPoint[1]], // 线段终点坐标（以圆心为起点，向上偏移半径长度）
          ];
        }
        const lineFeature = new Feature(new LineString(lineCoordinates));
        lineFeature.setStyle(lineStyle);
        let metersPerUnit = map.getView().getProjection().getMetersPerUnit();
        let circleRadius = params.range / metersPerUnit;
        const circle = createRegularPolygonCurve(fromLonLat([120.29340255, 29.3125972]), circleRadius, 100, params.angle, angle);
        var fanFeature = new Feature(circle);
        fanFeature.setStyle(fanStyle);
        if (cameraVectorLayer.getSource().getFeatures().length >= 2) {
          cameraVectorLayer.getSource().clear();
        }
        // 添加线段到图层
        cameraVectorLayer.getSource().addFeature(lineFeature);
        cameraVectorLayer.getSource().addFeature(fanFeature);
      }
    }

    function OrientToAngle(orient) {
      let tangle = -(orient - 90)
      if (tangle < 0.0) {
        while (tangle < 0.0) tangle += 360;
      }
      return tangle
    }

    function drawPoint() {
      if (lonlat != '') {
        let x = lonlat.coords[0].x + 0;
        let y = lonlat.coords[0].y + 0;
        let h = lonlat.coords[0].h + 0;
        let bearing = lonlat.bearing;
        let range = lonlat.range;
        x = Math.floor(x * 100000000) / 100000000;
        y = Math.floor(y * 100000000) / 100000000;
        h = Math.floor(h * 100) / 100;
        bearing = Math.floor(bearing * 100) / 100;
        range = Math.floor(range * 100) / 100;
        const pointStyle = new Style({
          image: new Icon({
            src: 'point.png',
            anchor: [0.5, 1]
          }),
          text: new Text({
            font: 'normal 12px 黑体',
            textAlign: 'center',
            textBaseline: 'middle',
            offsetY: -25,
            offsetX: 0,
            backgroundFill: new Stroke({
              color: 'rgba(0, 0, 255, 0.7)'
            }),
            fill: new Fill({
              color: 'rgba(236, 218, 20, 1)'
            }),
            padding: [5, 5, 5, 5],
            text: '经度:' + x + ', 纬度:' + y + ', 高程:' + h + '\r\n' + '方位:' + bearing + ', 距离:' + range
          })
        });

        const pointFeature = new Feature({
          geometry: new Point(fromLonLat([lonlat.coords[0].x, lonlat.coords[0].y]))
        });
        pointFeature.setStyle(pointStyle);
        if (pointVectorLayer.getSource().getFeatures().length >= 1) {
          pointVectorLayer.getSource().clear();
        }
        pointVectorLayer.getSource().addFeature(pointFeature);
      }
    }

    function diffDistance() {
      const length = getDistance(start, end)
      return length < 1000 ? Math.round(length) + '米' : Math.round(length / 1000) + '公里'
    }

    function createRegularPolygonCurve(origin, radius, sides, r, angle) {
      let rotation = 360 - r;
      radius = radius * 1.15;
      let angleStep = Math.PI * ((1 / sides) - (1 / 2))
      if (rotation) {
        angleStep += (rotation / 180) * Math.PI
      }
      let rotatedAngle, x, y
      let points = []
      for (let i = 0; i < sides; i++) {
        let angleOffset = i * ((360 - rotation) / 360)
        rotatedAngle = angleStep + (angleOffset * 2 * Math.PI / sides)
        x = origin[0] + (radius * Math.cos(rotatedAngle));
        y = origin[1] + (radius * Math.sin(rotatedAngle));
        points.push([x, y]);
      }
      if (rotation != 0) {
        points.push(origin);
      }
      var ring = new LinearRing(points);
      ring.rotate((angle + 90 + 1 / 2 * r) / 57.3, origin);
      let list = ring.getCoordinates();

      return new Polygon([list]);
    }

    async function fetchData() {
      try {
        const response = await axios.get('http://8.148.10.46:3050/api/MapComp');
        WarnPoint.splice(0, WarnPoint.length, ...response.data);
        console.log('WarnPoint', WarnPoint);
        WarnPoint.forEach(point => {
          const iconFeature = new Feature({
            geometry: new Point(fromLonLat([point.cenlon, point.cenlat]))
          });
          iconFeature.setStyle(createIconStyle());
          warnPointVectorSource.addFeature(iconFeature)
        });
      } catch (error) {
        console.error('Error fetching data:', error);
      }
    }//获取告警点位并添加

    function createIconStyle() {
      return new Style({
        image: new Icon({
          anchor: [0.5, 1], // 图标的锚点设置为底部中心
          anchorXUnits: 'fraction',
          anchorYUnits: 'fraction',
          src: './public/img/warning.png', // 图标的URL，确保正确指向图标文件
          scale: 1 // 根据图标的实际大小调整比例
        })
      });
    }

    function updateCenterFeature(newCoordinate) {
      if (!newCoordinate) return;

      // Convert coordinate to map projection
      const olCoordinate = fromLonLat(newCoordinate);
      const feature = new Feature(new Point(olCoordinate));
      CenterVectorSource.clear(); // Clear previous feature
      CenterVectorSource.addFeature(feature); // Add new feature
    }

    function updateCameraFeature(points) {
      points.forEach(point => {
        const CameraPoints = fromLonLat(point);
        const feature = new Feature(new Point(CameraPoints));
        CameraVectorSource.addFeature(feature);
      });
    }

    function switchToImg() {
      map.getLayers().setAt(0, new TileLayer({
        source: new XYZ({
          url: 'https://t6.tianditu.gov.cn/img_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=img&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILECOL={x}&TILEROW={y}&TILEMATRIX={z}&tk=22d8b67fcba918adc0a566e96d1587b3'
        })
      }));
    }

    // 切换至矢量图层
    function switchToVec() {
      map.getLayers().setAt(0, new TileLayer({
        source: new XYZ({
          url: 'https://t6.tianditu.gov.cn/vec_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=vec&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILECOL={x}&TILEROW={y}&TILEMATRIX={z}&tk=22d8b67fcba918adc0a566e96d1587b3'
        })
      }));
    }


    return {
      socket_data,
      curve,
      lonlat,
      map,
      switchToImg,
      switchToVec,
    }
  },
}
</script>

<template>
  <div id="map">
    <div class="btn-group">
      <a-button class="btn" @click="switchToImg">影像</a-button>
      <a-button class="btn" @click="switchToVec">矢量</a-button>
    </div>
    <div id="legend">
      <h2 class="legend-title">图例</h2>
      <div class="legend-item">
        <img src="/img/camera.png">
        <h3>摄像头</h3>
      </div>
      <div class="legend-item">
        <img src="/img/warning.png">
        <h3>告警点位</h3>
      </div>
    </div>
  </div>
</template>

<style scoped>
#map {
  width: 100%;
  height: 100%;
}
.btn-group {
  position: absolute;
  top: 2%;
  right: 5%;
  z-index: 1000;
}
.btn {
  height: 6.5vh;
  width: auto;
  margin-right: 2vh;
  cursor: pointer;
  padding: 1vh 1vh;
  background-color: #3ea164;
  color: #e1e0e0;
  border: none;
  border-radius: 1vh;
  font-weight: bold;
  font-size: 2vh;
}
.btn:hover {
  background-color: #cdeec2;
  color: #262525;
}
#legend {
  height: 35vh;
  width: 25vh;
  background-color: #c6debd;
  position: absolute;
  bottom: 0.5%;
  right: 0.5%;
  opacity: 0.8;
  z-index: 1000;
  flex-direction: column;
  border-radius: 1vh;
}
.legend-title {
  color:#1c1c1c;
  font-weight:bold;
  font-size: 3.2vh;
  margin-top: 2vh;
  padding-left: 1vh;
}
.legend-item {
  display: flex;
  align-items: center;
  padding-left: 2vh;
  margin-bottom: 1vh;
  img {
    margin-right: 1vh;
  }
  h3 {
    font-size: 2.5vh;
    font-weight: bold;
    color: #1c1c1c;
  }
}
</style>