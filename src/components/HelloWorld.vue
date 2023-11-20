<template>
  <div>
    <div id="map-container"></div>
  </div>
</template>

<script>
import AMapLoader from '@amap/amap-jsapi-loader'

window._AMapSecurityConfig = {
  securityJsCode: 'eceded719ab73aba9455d1a2fdb4a008'
}
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  data() {
    return {
      map: null,
      AMap: null,
      spinning: false
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
        console.log('11111111111111111111')
      AMapLoader.load({
        key: '13cd07438f4a833b2156faeacb78e5ec',
        version: '1.4.15',
        plugins: ['AMap.MarkerClusterer']
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
              })
            ]
          })

          this.map.on('complete', () => {
            // 地图图块加载完成后触发
            this.spinning = false
          })
          resolve(AMap)
        })
        .catch((e) => {
          console.log(e)
          reject(e)
        })
      })
    }
  }
}
</script>

<style scoped>
#map-container {
  width: 1920px;
  height: 100vh;
}
</style>
