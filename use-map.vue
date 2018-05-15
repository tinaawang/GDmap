<template>
  <my-map ref="map" @select="select"></my-map>
</template>
<script>
import myMap from '../../components/my-map/myMap';
export default{
  name:'',

  data(){
    return{}
  },
  methods:{// 手动选择位置
    select(loc) {
      // 处理经纬度

      this.formData.lng = loc.lng;
      this.formData.lat = loc.lat;
    },
     async openMap(province, address) {
      let keyword = `${province}${address}`;

      const res = await this.$refs['map'].search(keyword);
      if (res) {
        this.formData.lng = res.location.lng;
        this.formData.lat = res.location.lat;
      }

      // this.mapAddress = res.location;
      console.info(res);

      this.$refs['map'].open();
      this.text = '已标记地图位置';
    },
  },
    }

},

</script>
