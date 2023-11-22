<template>
  <div class="map-page">
    <div class="map-btn">
      <a-button type="primary">绘制圆形</a-button>
      <a-button type="primary">绘制矩形</a-button>
      <a-button type="primary">绘制多边形</a-button>
    </div>
    <div class="item-map data-info">
      <p>当前所在行政区信息</p>
      <vue-json-pretty :data="dataInfo" :highlightMouseoverNode ="true"></vue-json-pretty>
    </div>
    <div class="item-map map-styles">
      <p>官方默认自定义样式</p>
      <a-radio-group>
        <div v-for="(item, index) in styleList" :key="index">
          <a-radio :value="item.value" @click="handleSelectStyle(item.value)"></a-radio>
          <span>{{ item.label }}</span>
          <span>{{ item.value }}</span>
        </div>
      </a-radio-group>
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
      dataInfo: null,
      styleList: [
        { value: 'normal', label: '标准' },
        { value: 'dark', label: '幻影黑' },
        { value: 'light', label: '月光银' },
        { value: 'whitesmoke', label: '远山黛' },
        { value: 'fresh', label: '草色青' },
        { value: 'grey', label: '雅士灰' },
        { value: 'graffiti', label: '涂鸦' },
        { value: 'macaron', label: '马卡龙' },
        { value: 'blue', label: '靛青蓝' },
        { value: 'darkblue', label: '极夜蓝' },
        { value: 'wine', label: '酱籽' },
      ],
      marker: null
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
            pitch: 90,
            rotation: 90,
            rotateEnable: true, // 是否可以旋转
            pitchEnable: true, // 是否可以倾斜
            showBuildingBlock: true,
            buildingAnimation: true,
            // showLabel: false, //不显示地图文字标记
            layers: [
              new AMap.TileLayer(), // 高德默认标准图层
              // new AMap.TileLayer.Satellite(), // 卫星图层
              // new AMap.TileLayer.RoadNet(), // 路网图层
              // new AMap.TileLayer.Traffic(), // 实时路况图层
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
          this.marker = new AMap.Marker({
            icon: 'https://webapi.amap.com/theme/v1.3/markers/n/mark_b.png',
            position: [113.072972, 28.222085],
            draggable: true
          })
          this.map.addControl(toolBar)
          this.map.addControl(controlBar)
          this.map.add(this.marker)
        
          this.map.on('complete', () => {
            // 地图图块加载完成后触发
            this.spinning = false
            this.logMapInfo()
            // this.map.setCity('北京市') // 设置地图当前行政区
          })
          this.map.on('moveend', (e) => {
            this.logMapInfo(e)
          })
          this.map.on('click', (e) => {
            this.handleMapClick(e)
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
    logMapInfo(e){
      const center = this.map.getCenter() // 获取当前地图级别
      const { lat, lng } = center
      console.log('moveend===========', e, lat, lng)
      this.map.getCity((info) => {
        this.dataInfo = info
      });
    },
    // 获取鼠标点击经纬度
    handleMapClick(e) {
      const { lat, lng } = e.lnglat
      console.log('e-lat-lng=============', lat, lng)
      if (this.marker) {
        this.map.remove(this.marker)
      }
      this.marker = new this.AMap.Marker({
        icon: 'https://webapi.amap.com/theme/v1.3/markers/n/mark_b.png',
        position: [lng, lat],
        draggable: true
      })
      this.map.add(this.marker)
      this.marker.on('click', (e) => {
        this.handleInfoWindow(e)
      })
      this.logMapInfo(e)
    },
    handleInfoWindow(e) {
      console.log(e)
      const { lat, lng } = e.lnglat
      const infoWindow = new this.AMap.InfoWindow({
        content: `
          <div>经度: ${lng}</div>
          <div>纬度: ${lat}</div>
        `
      })
      infoWindow.on('open')
      infoWindow.open()
    },
    // 设置样式主题
    handleSelectStyle(value) {
      const styleName = `amap://styles/${value}`
      this.map.setMapStyle(styleName)
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
    top: 20px;
    left: 20px;
  }
  .map-styles {
    bottom: 20px;
    right: 20px;
    height: 300px;
    span {
      display: inline-block;
      width: 50px;
      margin-right: 20px;
    }
  }
  #map-container {
    width: 1920px;
    height: 100vh;
  }
}
</style>
