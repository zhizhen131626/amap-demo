<template>
  <div class="map-page">
    <div class="map-btn">
      <a-button type="primary">绘制圆形</a-button>
      <a-button type="primary">绘制矩形</a-button>
      <a-button type="primary">绘制多边形</a-button>
    </div>
    <div class="data-info">
      <p>当前所在行政区信息</p>
      <vue-json-pretty :data="dataInfo" :highlightMouseoverNode ="true"></vue-json-pretty>
    </div>
    <div id="map-container"></div>
  </div>
</template>

<script>
import AMapLoader from '@amap/amap-jsapi-loader'
import VueJsonPretty from 'vue-json-pretty'
import 'vue-json-pretty/lib/styles.css'

window._AMapSecurityConfig = {
  securityJsCode: 'eceded719ab73aba9455d1a2fdb4a008'
} 
export default {
  name: 'HelloWorld',
  components: { VueJsonPretty },
  props: {
    msg: String
  },
  data() {
    return {
      map: null,
      AMap: null,
      spinning: false,
      dataInfo: null
    }
  },
  mounted() {
    this.initMap().then((AMap) => {
      this.AMap = AMap
    })
  },
  methods: {
    initMap() {
      return new Promise((resolve, reject) => {
        AMapLoader.load({
          key: '13cd07438f4a833b2156faeacb78e5ec',
          version: '1.4.15',
          plugins: ['AMap.MarkerClusterer', 'AMap.ToolBar', 'AMap.ControlBar']
        }).then((AMap) => {
          // 初始化3D地图并设置中心点坐标和地图级别
          this.map = new AMap.Map('map-container', {
            viewMode: '3D',
            center: [113.072972, 28.222085],
            mapStyle: 'amap://styles/8479f384cd3bc8395765f8bb5221f00c',
            zoom: 15,
            pitch: 40,
            showBuildingBlock: true,
            buildingAnimation: true,
            layers: [
              // 高德默认标准图层
              new AMap.TileLayer(),
              // 楼块图层
              new AMap.Buildings({
                zooms: [3, 18],
                zIndex: 10,
                heightFactor: 2
              }),
            ]
          })
          const toolBar = new AMap.ToolBar({
            visible: false,
            position: {
              top: '110px',
              right: '40px'
            }
          })
          const controlBar = new AMap.ControlBar({
            visible: false,
            position: {
              top: '10px',
              right: '10px'
            }
          })
          this.map.addControl(toolBar)
          this.map.addControl(controlBar)
        
          this.map.on('complete', () => {
            // 地图图块加载完成后触发
            this.spinning = false
            this.logMapInfo()
          })
          this.map.on('moveend', () => {
            // 地图图块加载完成后触发
            this.logMapInfo()
          })
          resolve(AMap)
        })
        .catch((e) => {
          console.log(e)
          reject(e)
        })
      })
    },
    //获取并展示当前城市信息
    logMapInfo(){
      this.map.getCity((info) => {
        this.dataInfo = info
      });
    }
  }
}
</script>

<style lang="less" scoped>
.map-page {
  position: relative;
  .map-btn {
    position: absolute;
    top: 20px;
    left: 50%;
    transform: translate(-50%, 0);
    z-index: 999;
    .ant-btn {
      border-radius: 0;
    }
    .ant-btn:not(:last-child) {
      border-right-color: #eee;
    }
  }
  .data-info {
    position: absolute;
    width: 200px;
    height: 200px;
    top: 20px;
    left: 20px;
    z-index: 999;
    background-color: #fff;
    padding: 10px;
    // text-align: center;
  }
  #map-container {
    width: 1920px;
    height: 100vh;
  }
}
</style>
