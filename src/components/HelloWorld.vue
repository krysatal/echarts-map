<style scoped>
  .container {
    /*position: absolute;*/
    /*width: calc(100%);*/
    /*height: calc(100%);*/
    /*top: 0;*/
    width: 100%;
    height: 100%;
    background: url("../../static/img/bg.png") no-repeat;
    background-size: 100% 100%;
  }

  .container #main {
    width: 100%;
    min-height: 100%;
  }
</style>

<template>
  <div class="container">
    <div id="main" class="myCharts" ref="myCharts"></div>
  </div>
</template>

<script>
  import eCharts from 'echarts'
  import 'echarts/extension/bmap/bmap'
  import china from './china'
  import nameMap from './nameMap'

  let myCharts = null
  export default {
    data () {
      return {
        // myCharts: '',
        BMap: null,
        geoCoordMap: {
          '浙江省': [120.153576, 30.287459],
          '北京市': [116.405285, 39.904989],
          '河北省': [114.502461, 38.045474],
          '山西省': [112.549248, 37.857014],
          '辽宁省': [123.429096, 41.796767],
          '上海市': [121.438742, 31.072555],
          '山东省': [117.000923, 36.675807],
          '广东省': [113.280637, 23.125178],
          '西藏自治区': [91.132212, 29.660361]
        },
        commonOption: {
          color: '#fff464',
          backgroundColor: 'transparent',
          tooltip: {
            show: true,
            formatter: '{b} <br/> {c}'
          },
          visualMap: {
            type: 'piecewise',
            splitNumber: 10,
            min: 0,
            max: 1000,
            text: ['High', 'Low'],
            realtime: false,
            calculable: true,
            inRange: {
              color: ['#e0ffff', '#006edd']
            },
            show: false
          },
          grid: {
            top: 0,
            bottom: 0,
            left: 0,
            right: 0
          },
        }
      }
    },
    beforeDestroy () {
      if (!myCharts) {
        return false
      }
      window.removeEventListener("resize", this.resize)
      window.removeEventListener('beforeunload', this.clearChart)
      myCharts.dispose()
      myCharts = null
    },
    mounted () {
      const self = this
      this.computedWindowSize()
      window.addEventListener('beforeunload', this.clearChart)
      let _option = Object.assign(self.commonOption, self.getChinaOption('china', china))
      this.initEcharts('china', china, _option, 'province')
    },
    methods: {
      clearChart () {
        this.$destroy()
      },
      removeEventListener () {
        if (!myCharts) {
          return false
        }
        myCharts.dispose()
        myCharts = null
      },
      /***
       * @method 初始化地图
       * @param mapType 地图名 china/zhejiang/hangzhou
       * @param mapJson 地图json
       * @param mapOption 地图配置项
       * @param type 地图类型 china/province...
       */
      initEcharts (mapType, mapJson, mapOption, type) {
        const self = this
        if (document.getElementById('main') && document.getElementById('main').hasAttribute('_echarts_instance_')) {
          document.getElementById('main').removeAttribute('_echarts_instance_')
          document.getElementById('main').innerHTML = ''
        }
        myCharts = eCharts.init(document.getElementById('main'))
        eCharts.registerMap(mapType, mapJson)
        myCharts.setOption(mapOption)
        if (type === 'province') {
          myCharts.on('click', function (params) {
            self.getProvinceInfo(params, type)
          })
        } else if (type === 'city') {
          myCharts.on('contextmenu', (params) => {
            let _option = Object.assign(self.commonOption, self.getChinaOption('china', china))
            self.removeEventListener()
            self.initEcharts('china', china, _option, 'province')
          })
        }
      },
      /****
       * @method 生成全国地图的配置项
       * */
      getChinaOption (mapType, mapJson) {
        const self = this
        return {
          geo: {
            show: true,
            map: mapType,
            // zoom: 1.2,
            roam: false,
            itemStyle: {
              borderColor: '#006edd',
              borderWidth: 5,
              shadowBlur: 10,
              shadowColor: '#e0ffff',
              opacity: 0.3
            },
            layoutCenter: ['72%', '60%'],
            layoutSize: 1000
          },
          // bmap: {
          //   zoom: 5,
          //   roam: false
          // },
          series: [
            {
              name: '测试数据',
              type: 'map',
              // zoom: 1.2,
              map: mapType,
              aspectScale: 0.75,
              roam: false,
              layoutCenter: ['72%', '60%'],
              layoutSize: 1000,
              // itemStyle: {
              //   borderColor: 'red'
              // },
              data: self.getSeriesInfo(mapJson)
            },
            {
              name: 'Top',
              type: 'scatter',
              coordinateSystem: 'geo',
              encode: {
                value: 2
              },
              symbolSize: 10,
              showEffectOn: 'render',
              rippleEffect: {
                brushType: 'stroke'
              },
              hoverAnimation: true,
              label: {
                formatter: '{b}',
                position: 'right',
                show: true
              },
              itemStyle: {
                color: '#fff464',
                shadowBlur: 10,
                shadowColor: '#333'
              },
              data: self.convertData([{
                name: '浙江省',
                value: self.getRandomData()
              }, {
                name: '北京市',
                value: self.getRandomData()
              }, {
                name: '河北省',
                value: self.getRandomData()
              }, {
                name: '山西省',
                value: self.getRandomData()
              }, {
                name: '辽宁省',
                value: self.getRandomData()
              }, {
                name: '山东省',
                value: self.getRandomData()
              }, {
                name: '广东省',
                value: self.getRandomData()
              }, {
                name: '西藏自治区',
                value: self.getRandomData()
              }])
            },
            {
              name: 'Top 5',
              type: 'effectScatter',
              coordinateSystem: 'geo',
              encode: {
                value: 2
              },
              showEffectOn: 'render',
              rippleEffect: {
                brushType: 'stroke'
              },
              hoverAnimation: true,
              label: {
                formatter: '{b}',
                position: 'right',
                show: true
              },
              itemStyle: {
                color: '#fff464',
                shadowBlur: 10,
                shadowColor: '#333'
              },
              data: self.convertData([{
                name: '北京市',
                value: self.getRandomData()
              }, {
                name: '上海市',
                value: self.getRandomData()
              }])
            }
          ]
        }
      },
      /****
       * @method 生成全省地图的配置项
       * */
      getProvinceOption (mapType, mapJson) {
        const self = this
        return {
          geo: {
            show: true,
            map: mapType,
            // zoom: 1.2,
            roam: false,
            itemStyle: {
              borderColor: '#006edd',
              borderWidth: 5,
              shadowBlur: 10,
              shadowColor: '#e0ffff',
              opacity: 0.3
            },
            layoutCenter: ['72%', '50%'],
            layoutSize: 800
          },
          series: [{
            name: '测试数据',
            type: 'map',
            map: mapType,
            layoutCenter: ['72%', '50%'],
            layoutSize: 800,
            aspectScale: 0.75,
            roam: false,
            data: self.getSeriesInfo(mapJson)
          }]
        }
      },
      /****
       * @method 生成每个省/市...的假数据
       * */
      getSeriesInfo (json) {
        const self = this
        return json.features.map(province => {
          return {
            name: province.properties.name,
            value: self.getRandomData()
          }
        })
      },
      /****
       * @method 钻到省级
       * */
      getProvinceInfo (params, type) {
        const self = this
        // 引入vue-resource 才能使用this.$http
        this.$http.get('static/' + type + '/' + nameMap[type][params.name] + '.json').then(function (data) {
          eCharts.registerMap(nameMap[type][params.name], data.body)
          let _option = Object.assign(self.commonOption, self.getProvinceOption(nameMap[type][params.name], data.body))
          self.removeEventListener()
          self.initEcharts(nameMap[type][params.name], data.body, _option, 'city')
        })
      },
      /****
       * @method 钻到市级
       * */
      getCityInfo (params) {
        console.log(params)
      },
      /****
       * @method 生成每个省的数据
       * @return {number}
       */
      getRandomData () {
        return Math.round(Math.random() * 1000)
      },
      /****
       * @method 封装echarts散点数据格式
       * @param data
       * @return {Array}
       */
      convertData (data) {
        const self = this
        let res = []
        for (let i = 0; i < data.length; i++) {
          let geoCoord = self.geoCoordMap[data[i].name]
          if (geoCoord) {
            res.push({
              name: data[i].name,
              value: geoCoord.concat(data[i].value),
              visualMap: false
            })
          }
        }
        return res
      },
      /***
       * @method 计算出窗口容器的大小
       */
      computedWindowSize () {
        this.$refs.myCharts.style.height = window.innerHeight + 'px'
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
