<!-- 散点图 -->
<template lang="html">
<div class="heat">
  <v-header :name="name" :legendArr="legendArr" :myChart="myChart"></v-header>
  <div class="main">
    <!-- <el-amap vid="amapDemo"  :center="center" :zoom="zoom" :events="events" class="amap-demo" :zoomEnable="zoomEnable"></el-amap> -->
    <div id="amap-main"></div>
    <div>
      <!-- <input type="button" class="button" value="开始" id="start" style="position:absolute;top:10px;right:200px"/>
      <input type="button" class="button" value="释放" id="pause" style="position:absolute;top:10px;right:150px"/>
      <input type="button" class="button" value="回收" id="resume" style="position:absolute;top:10px;right:100px"/>
      <input type="button" class="button" value="暂停" id="stop" style="position:absolute;top:10px;right:50px"/>
      <input type="button" class="button" value="继续" id="load" style="position:absolute;top:10px;right:0px"/> -->
      <input type="button" class="button" value="GO" id="load" style="position:absolute;top:10px;right:0px" @click="stepByStep()"/>
    </div>
  </div>
</div>
</template>

<script>
import axios from 'axios'
import echarts from 'echarts'
import china from 'echarts/map/js/china'
import header from 'components/header/header'
import { lazyAMapApiLoaderInstance } from 'vue-amap';
// let amapManager = new VueAMap.AMapManager();
let _this = this;
module.exports = {
  data: function() {
    return {
      userStatus: 'start',
      isEmpty: true,
      rbtReady: 0,
      map: null,
      data: null,
      car: null,
      step: 0,
      robot: [],
      legendArr: [],
      lineArr: [
        // [121.468882, 31.214489],
        // [121.468174, 31.214268],
        // [121.4676, 31.215218],
        // [121.467203, 31.216113],
        // [121.464821, 31.215535],
        // [121.46458, 31.216507]
        // [121.466758, 31.2172],
        // [121.46635, 31.2182],
        // [121.467514, 31.21865],
        // [121.468775, 31.219214],
        // [121.469376, 31.219388],
        // [121.469756, 31.219384],
        // [121.470631, 31.219347],
        // [121.470813, 31.219159],
        // [121.470931, 31.21904],
        // [121.471446, 31.218989],
        // [121.471725, 31.219058],
        // [121.471993, 31.21915],
        // [121.473141, 31.21943],
        // [121.473168, 31.218732],
        // [121.473195, 31.21804],
        // [121.473262, 31.216306],
        // [121.473327, 31.21571],
        // [121.474485, 31.216104]
      ],
      color: this.$store.state.color,
      myChart: {},
      name: '总控台',
      zoom: 19,
      zoomEnable: true,
      center: [121.468812, 31.214569],
      events: {
        init(o) {
          let car = null;
          let robot = [];
          let lineArr = [
            [121.468882, 31.214489],
            [121.468174, 31.214268],
            [121.4676, 31.215218],
            [121.467203, 31.216113],
            [121.464821, 31.215535],
            [121.46458, 31.216507]
            // [121.466758, 31.2172],
            // [121.46635, 31.2182],
            // [121.467514, 31.21865],
            // [121.468775, 31.219214],
            // [121.469376, 31.219388],
            // [121.469756, 31.219384],
            // [121.470631, 31.219347],
            // [121.470813, 31.219159],
            // [121.470931, 31.21904],
            // [121.471446, 31.218989],
            // [121.471725, 31.219058],
            // [121.471993, 31.21915],
            // [121.473141, 31.21943],
            // [121.473168, 31.218732],
            // [121.473195, 31.21804],
            // [121.473262, 31.216306],
            // [121.473327, 31.21571],
            // [121.474485, 31.216104]
          ];
          let userArr = [
            [[121.467423, 31.214727]],
            [[121.467144, 31.214727]],
            [[121.467294, 31.214525]],
            [[121.467922, 31.215819]],
            [[121.467686, 31.215856]],
            [[121.465867, 31.216324]],
            [[121.466211, 31.216512]],
            [[121.465626, 31.216461]],
            [[121.463571, 31.215998]],
            [[121.464054, 31.215888]],
            [[121.465411, 31.217507]],
            [[121.465819, 31.217466]],
            [[121.465304, 31.2172]],
            [[121.46804, 31.219393]],
            [[121.46789, 31.219205]],
            [[121.468399, 31.219411]],
            [[121.473286, 31.220306]],
            [[121.473554, 31.221113]],
            [[121.473941, 31.219095]],
            [[121.47511, 31.219177]],
            [[121.473651, 31.218443]],
            [[121.472321, 31.21692]],
            [[121.471999, 31.217462]],
            [[121.474681, 31.217205]]
          ];
          let stopArr = [
            [[121.468629, 31.21441], 0],
            [[121.467859, 31.21479], 1],
            [[121.467433, 31.215594], 1],
            [[121.467213, 31.21609], 0],
            [[121.46464, 31.216264], 1],
            [[121.466189, 31.215867], 1],
            [[121.465827, 31.216904], 1],
            [[121.467813, 31.218784], 1],
            [[121.469961, 31.219375], 0],
            [[121.47282, 31.219352], 1],
            [[121.47317, 31.218678], 2],
            [[121.473223, 31.217317], 1],
            [[121.474422, 31.216083], 1]
          ];
          let polyline, passedPolyline;
          let dist = [];
          o.plugin('AMap.Geolocation', function() {
            // let geolocation = new AMap.Geolocation({
            //   enableHighAccuracy: true,
            //   timeout: 10000,
            //   buttonOffset: new AMap.Pixel(10, 20),
            //   zoomToAccuracy: true,
            //   buttonPosition: 'RB'
            // });
            // o.addControl(geolocation);
            // geolocation.getCurrentPosition();
            // AMap.event.addListener(geolocation, 'complete', function(data) {
              // console.log(data);

            var location = [121.468812, 31.214569];
            // location[0] = data.position.getLng();
            // location[1] = data.position.getLat();

            lineArr.unshift(location);

            var iconCar = new AMap.Icon({
              image: 'http://opub24jup.bkt.clouddn.com/ico_car_1.png',
              size: new AMap.Size(39.5, 28.5),
              imageSize: new AMap.Size(39.5, 28.5)
            });
            car = new AMap.Marker({
              map: o,
              position: location,
              icon: iconCar,
              offset: new AMap.Pixel(-18, -14),
              autoRotation: true,
              zIndex: 100
            });
            lineArr.unshift(location);
            var polyline = new AMap.Polyline({
              map: o,
              path: lineArr,
              strokeColor: "#FFB63E",
              // strokeOpacity: 1,
              strokeWeight: 4
              // strokeStyle: "solid"
            });
            var passedPolyline = new AMap.Polyline({
              map: o,
              // path: lineArr,
              strokeColor: "#0E92FF",
              // strokeOpacity: 1,
              strokeWeight: 4
              // strokeStyle: "solid"
            });

            AMap.event.addDomListener(document.getElementById('start'), 'click', function() {
              car.on('moving', function(e) {
                passedPolyline.setPath(e.passedPath);
              })
              o.setFitView();
              car.moveAlong(lineArr, 50);
            }, false);

            let brand = [
              'http://opub24jup.bkt.clouddn.com/coos/pic_bsk.png',
              'http://opub24jup.bkt.clouddn.com/coos/pic_hmxs.png',
              'http://opub24jup.bkt.clouddn.com/coos/pic_mdl.png',
              'http://opub24jup.bkt.clouddn.com/coos/pic_xbk.png',
              'http://opub24jup.bkt.clouddn.com/coos/pic_xysj.png'
            ];
            let i = 0;
            let rbtLine = [
              [[121.468321, 31.214592]],
              [[121.467849, 31.213968]],
              [[121.468193, 31.216344], [121.467796, 31.216271]],
              [[121.470285, 31.218904]],
              [[121.470596, 31.219647]],
              [[121.469673, 31.218757]],
              [[121.473498, 31.218088]]];
            let idx = 0;
            rbtLine.forEach(function(rbt) {
              rbt.forEach(function(loc) {
                idx++;
                if (idx >= brand.length - 1) idx = 0;
                var icon = new AMap.Icon({
                  image: brand[idx],
                  size: new AMap.Size(18, 18),
                  imageSize: new AMap.Size(18, 18)
                });
                var iconStore = new AMap.Marker({
                  map: o,
                  position: loc,
                  icon: icon,
                  offset: new AMap.Pixel(-9, -9),
                  autoRotation: true,
                  zIndex: 100
                });
              })
            })
            let idy = 0;
            userArr.forEach(function(rbt) {
              rbt.forEach(function(loc) {
                idy++;
                if (idy >= brand.length - 1) idy = 0;
                var icon = new AMap.Icon({
                  image: 'http://opub24jup.bkt.clouddn.com/coos/boton_dwd_yellow.png',
                  size: new AMap.Size(18, 18),
                  imageSize: new AMap.Size(18, 18)
                });
                var iconStore = new AMap.Marker({
                  map: o,
                  position: loc,
                  icon: icon,
                  offset: new AMap.Pixel(-9, -9),
                  autoRotation: true,
                  zIndex: 100
                });
              })
            })

            stopArr.forEach(function(rbt) {
              var icon = new AMap.Icon({
                image: 'http://opub24jup.bkt.clouddn.com/coos/boton_dwd.png',
                size: new AMap.Size(18, 18),
                imageSize: new AMap.Size(18, 18)
              });
              var iconStore = new AMap.Marker({
                map: o,
                position: rbt[0],
                icon: icon,
                offset: new AMap.Pixel(-9, -9),
                autoRotation: true,
                zIndex: 100
              });
            })


            AMap.event.addDomListener(document.getElementById('pause'), 'click', function() {
              car.pauseMove();
              let position = car.getPosition();
              rbtLine[i].unshift([position.lng, position.lat]);
              // rbtLine[i].push(dist[i]);
              // rbtLine[i].push([position.lng, position.lat]);
              let rbt = new AMap.Marker({
                map: o,
                position: [position.lng, position.lat],
                icon: "http://opub24jup.bkt.clouddn.com/coos/ico_robot_small.png",
                offset: new AMap.Pixel(-36, -28),
                autoRotation: false
              });

              var polylineRbt = new AMap.Polyline({
                map: o,
                path: rbtLine[i],
                strokeColor: "#677C96",
                strokeOpacity: 0,
                strokeWeight: 4,
                strokeStyle: "dashed"
              });
              var passedPolylineRbt = new AMap.Polyline({
                map: o,
                strokeColor: "#677C96",
                strokeOpacity: 1,
                strokeWeight: 4,
                strokeStyle: "dashed"
              });
              rbt.on('moving', function(e) {
                passedPolylineRbt.setPath(e.passedPath);
              })
              o.setFitView();
              rbt.moveAlong(rbtLine[i], 80);
              robot.push(rbt);
              i++;
                // j++;
              // }
            }, false);
            AMap.event.addDomListener(document.getElementById('stop'), 'click', function() {
              let position = car.getPosition();
              console.log('释放点');
              console.log([position.lng, position.lat]);
              car.pauseMove();
            }, false);
            AMap.event.addDomListener(document.getElementById('load'), 'click', function() {
              let e = car.getPosition();
              car.resumeMove();
            }, false);
            AMap.event.addDomListener(document.getElementById('resume'), 'click', function() {
              robot.forEach((rbt) => {
                o.remove(rbt);
              })
              car.resumeMove();
            }, false);
            // });
            // AMap.event.addListener(geolocation, 'error', () => {});
            AMapUI.loadUI(['misc/MarkerList', 'overlay/SimpleMarker', 'overlay/SimpleInfoWindow'],
              function(MarkerList, SimpleMarker, SimpleInfoWindow) {
                var $ = MarkerList.utils.$;
                var dblClickEventListener = o.on('rightclick', function(e) {
                  console.log([e.lnglat.getLng(), e.lnglat.getLat()]);
                  dist.push([e.lnglat.getLng(), e.lnglat.getLat()]);
                  new SimpleMarker({
                    // iconLabel: '1',
                    iconStyle: {
                      src: 'http://opub24jup.bkt.clouddn.com/coos/boton_dwd_blue.png',
                      style: {
                        width: '38px',
                        height: '38px'
                      }
                    },
                    offset: new AMap.Pixel(-18, -130),
                    map: o,
                    // showPositionPoint: true,
                    position: [e.lnglat.getLng(), e.lnglat.getLat()],
                    zIndex: 100
                  });
                });
                var clickEventListener = o.on('click', function(e) {
                  lineArr.push([e.lnglat.getLng(), e.lnglat.getLat()]);
                  console.log([e.lnglat.getLng(), e.lnglat.getLat()]);
                  polyline = new AMap.Polyline({
                    map: o,
                    path: lineArr,
                    strokeColor: "#FFB63E",
                    // strokeOpacity: 1,
                    strokeWeight: 3
                    // strokeStyle: "solid"
                  });
                  passedPolyline = new AMap.Polyline({
                    map: o,
                    strokeColor: "#0E92FF",
                    // strokeOpacity: 1,
                    strokeWeight: 3
                    // strokeStyle: "solid"
                  });
                });
              });
          });
        }
      }
    };
  },
  components: {
    'v-header': header
  },
  mounted() {
    this.initRouter();
    this.stepByStep();
    lazyAMapApiLoaderInstance.load().then(() => {
      this.map = new AMap.Map('amap-main', {
        zoom: 19,
        zoomEnable: true,
        center: [121.468812, 31.214569]
      });

      var polyline = new AMap.Polyline({
        map: this.map,
        path: this.lineArr,
        strokeColor: "#FFB63E",
        // strokeOpacity: 1,
        strokeWeight: 4
        // strokeStyle: "solid"
      });
    });
  },
  methods: {
    add() {
    },
    onComplete(data) {
    },
    onError(data) {
      console.log('error');
    },
    initRouter() {
      let _this = this;
      axios.post('http://jack.linkersocks.com:4999/bigMap/get/runMap/1000').then((res) => {
        _this.data = JSON.parse(res.data.data);
        console.log(_this.data);
      });
    },
    stepByStep() {
      let _this = this;
      setInterval(() => {
        axios.post('http://jack.linkersocks.com:4999/bigQueue/isEmpty/MinMaxStep').then((res) => {
          _this.isEmpty = res.data;
          // console.log(_this.isEmpty);
          if (_this.isEmpty === false) {
            axios.post('http://jack.linkersocks.com:4999/bigQueue/poll/MinMaxStep').then((response) => {
              var query = response.data.data.substr(1, response.data.data.length - 2);
              console.log(query);
              eval(query);
            });
          }
        });
        axios.post('http://jack.linkersocks.com:4999/bigMap/get/MinMaxDeliveryStatus/user').then((res) => {
          var json = JSON.parse(res.data.data);

          if (json.status === 'start') {
            var iconStore = new AMap.Icon({
              image: "http://opub24jup.bkt.clouddn.com/coos/boton_dwd_red.png",
              size: new AMap.Size(18, 18),
              imageSize: new AMap.Size(18, 18)
            });
            _this.userStatus = new AMap.Marker({
              map: _this.map,
              position: [121.467423, 31.214727],
              icon: iconStore,
              offset: new AMap.Pixel(-9, -9),
              zIndex: 100
            });
          } else if (json.status === 'finish') {
            if (_this.userStatus != null) {
              _this.map.remove(_this.userStatus);
            }
          }
        });
      }, 500);
    },
    goToMap(o, data, step, line) {
      var _this = this;

      if (_this.car == null) {
        // 车
        var iconCar = new AMap.Icon({
          image: 'http://opub24jup.bkt.clouddn.com/ico_car_1.png',
          size: new AMap.Size(39.5, 28.5),
          imageSize: new AMap.Size(39.5, 28.5)
        });
        _this.car = new AMap.Marker({
          map: o,
          position: data[step].path[0].line[0],
          icon: iconCar,
          offset: new AMap.Pixel(-18, -14),
          autoRotation: true,
          zIndex: 100
        });
      }

      // 商家
      if (data[step].point !== undefined) {
        o.setFitView();
        data[step].point.forEach(point => {
          var iconStore = new AMap.Icon({
            image: point.pic,
            size: new AMap.Size(18, 18),
            imageSize: new AMap.Size(18, 18)
          });
          var markStore = new AMap.Marker({
            map: o,
            position: point.location,
            icon: iconStore,
            offset: new AMap.Pixel(-9, -9),
            zIndex: 100
          });
        })
      }

      if (data[step].type === 2) {
        o.setFitView();
        _this.robot.forEach(rbt => {
          o.remove(rbt);
        })
        this.rbtReady = 0;
        // 路线
        var polyline = new AMap.Polyline({
          map: o,
          path: data[step].path[0].line,
          strokeColor: "#FFB63E",
          strokeOpacity: 1,
          strokeWeight: 4,
          strokeStyle: "solid"
        });
        var passedPolyline = new AMap.Polyline({
          map: o,
          strokeColor: "#0E92FF",
          strokeOpacity: 1,
          strokeWeight: 4,
          strokeStyle: "solid"
        });
        _this.car.moveAlong(data[step].path[0].line, 80);
        _this.car.on('movealong', function(e) {
          _this.step = step + 1;
        });
      } else if (data[step].type === 1) {
        o.setFitView();
        let rbtNo = line || 0;
        // document.querySelectorAll('.botton_right')[rbtNo].style.backgroundColor = '#C0CCE1';
        // document.querySelectorAll('.botton_right')[rbtNo].innerHTML = '等待中';
        data[step].path[rbtNo].status = 1;
        let rbt = new AMap.Marker({
          map: o,
          position: data[step].path[rbtNo].line[0],
          icon: "http://opub24jup.bkt.clouddn.com/coos/ico_robot_small.png",
          offset: new AMap.Pixel(-28, -28),
          autoRotation: false
        });

        var polylineRbt = new AMap.Polyline({
          map: o,
          path: data[step].path[rbtNo].line,
          strokeColor: "#677C96",
          strokeOpacity: 1,
          strokeWeight: 4,
          strokeStyle: "dashed"
        });
        // var passedPolylineRbt = new AMap.Polyline({
        //   map: o,
        //   strokeColor: "#677C96",
        //   strokeOpacity: 0,
        //   strokeWeight: 4,
        //   strokeStyle: "dashed"
        // });
        rbt.on('movealong', function(e) {
          // document.querySelectorAll('.botton_right')[rbtNo].style.backgroundColor = '#16D6B4';
          // document.querySelectorAll('.botton_right')[rbtNo].innerHTML = '已返回';
          data[step].path[rbtNo].status = 2;
          _this.rbtReady ++;
          console.log(_this.rbtReady);
          if (_this.rbtReady === data[step].path.length) {
            _this.step = step + 1;
          }
        })
        rbt.moveAlong(data[step].path[rbtNo].line, 80);
        _this.robot.push(rbt);
      }
    }
  }
};

</script>

<style lang="stylus" scoped>
.heat
  height 800px
  .main
    height 565px
#amap-main
  height: 800px;
    
</style>
