<template>
  <div v-show="visible">

    <div id="map-container" class="map-container">

    </div>

    <div class="close-btn" @click="close">确定</div>
  </div>
</template>

<script>
// 父组件通过 select 获取到经纬度
// search 方法获取到地理位置

const BEIJING = [116.39, 39.9];  // 设置初始进入地图的位置
export default {
  name: 'amap',
  props: {
    // mapAddress: String,
    shouldSelect: {
      // 是否添加marker选择地址
      type: Boolean,
      default: true,
    },
  },
  data() {
    return {
      visible: false,
      // chooseId: 2,
    };
  },
  methods: {
    open() {
      this.visible = true;
    },
    close() {
      this.visible = false;
    },
    // 加载sdk
    loadSdk() {
      return new Promise((resolve, reject) => {
        if (!window.AMap) {
          const script = document.createElement('script');
          script.src = `http://webapi.amap.com/maps?v=1.4.6&key=${key}`;
          document.head.appendChild(script);
          script.onload = resolve;
          script.onerror = reject;
        } else {
          resolve();
        }
      });
    },
    // 初始化地图
    async initMap() {
      await this.loadSdk();
      this.map = new AMap.Map('map-container', {
        zoom: 16,  // 地图初始缩放的大小
        center: BEIJING,  // 初始中心在北京
      });

      // 手动选择
      if (this.shouldSelect) {
        AMap.event.addListener(this.map, 'click', ({ lnglat }) => {
          this.$emit('select', lnglat);
          this.addMarker(lnglat);
        });
      }
    },
    // 设置地图中心
    setCenter(pointer = BEIJING) {
      // this.map.setZoom(14);
      this.map.setCenter(pointer);
    },
    // 覆盖物
    addMarker(position = BEIJING) {
      // 设置居中
      this.setCenter(position);

      if (this.marker) {
        this.map.remove(this.marker);
      }

      // 添加marker
      this.marker = new AMap.Marker({
        position, // marker所在的位置
        map: this.map, // 创建时直接赋予map属性
        icon: new AMap.Icon({
          image: 'http://webapi.amap.com/images/0.png',
          size: new AMap.Size(36, 36),
          imageOffset: new AMap.Pixel(-0, -0),
        }),
      });
      this.marker.setMap(this.map);
    },
    // 搜索 return Promise<array>
    search(keyword) {
      return new Promise(resolve => {
        AMap.service('AMap.PlaceSearch', () => {
          const placeSearch = new AMap.PlaceSearch({
            pageSize: 5,
            pageIndex: 1,
            // city: '上海'
          });
          placeSearch.search(keyword, (status, result) => {
            // 搜索到结果
            if (result.info !== 'OK') {
              resolve(null);
              return;
            }
            // 结果列表
            const { poiList: { pois = [] } } = result;
            const res = pois[0];
            resolve(res);
            // 添加marker
            this.addMarker(res.location);
          });
        });
      });
    },
  },
  mounted() {
    this.initMap();
  },
};
</script>

<style lang='scss' scoped>
.map-container {
  width: 100%;
  z-index: 99;
  height: 100%;
  background-color: #fff;
  position: fixed;
  left: 0;
  top: 0;
}
.close-btn {
  z-index: 999;
  width: 80%;
  margin: 0 auto;
  height: 0.9rem;

  border-radius: 0.1rem;
  border: none;
  position: fixed;
  background-color: #52ab38;
  left: 50%;
  font-size: 0.3rem;
  color: #fff;
  text-align: center;
  line-height: 0.9rem;
  transform: translateX(-50%);
  top: 90%;
}
</style>
