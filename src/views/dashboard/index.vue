<template>
    <div>
        <el-card class="danmu-control-pannel" shadow="hover">
            <div slot="header" class="clearfix">
                <span>弹幕操作</span>
            </div>
            <el-button @click="playDanmu" type="primary" size="mini" plain>播放</el-button>
            <el-button @click="pauseDanmu" type="primary" size="mini" plain>暂停</el-button>
            <el-button @click="hideDanmu" type="primary" size="mini" plain>隐藏</el-button>
            <el-button @click="showDanmu" type="primary" size="mini" plain>显示</el-button>
        </el-card>
        <el-card class="layer-control-pannel" shadow="hover">
            <div slot="header" class="clearfix">
                <span>图层操作</span>
            </div>
            <el-form label-width="80px" size="mini">
                <el-form-item label="轨道图层">
                    <el-switch v-model="gridLayerShow" @change="toggleGridLayer"></el-switch>
                </el-form-item>
                <el-form-item label="POI图层">
                    <el-select v-model="currentLayer" @change="selectPOILayer">
                        <el-option label="旅游景点" value="scene"></el-option>
                        <el-option label="美食餐饮" value="food"></el-option>
                        <el-option label="休闲娱乐" value="fun"></el-option>
                    </el-select>
                </el-form-item>
            </el-form>
        </el-card>
        <template v-for="(danmu, index) in danmus">
            <vue-danmaku v-model="danmu.content" class="danmaku" ref="danmaku" loop use-slot :channels="1"
                :randomChannel="false" :style="{ top: `calc((100vh - 50px)/8*${index})` }"
                :speeds="200 - (danmu.content.length * 5)" isSuspend>
                <template slot="dm" slot-scope="{ index, danmu }">
                    <div class="danmaku-name" v-if="danmu.name">
                        <wordcloud :mainWord="{ text: danmu.name, weight: 10 + Math.ceil((danmu.overall_rating - 4) * 10) }"
                            :keywordsList="danmu.tagList">
                        </wordcloud>
                    </div>
                </template>
            </vue-danmaku>
        </template>

        <div id="map-container"> </div>
    </div>
</template>

<script>
import 'mapbox-gl/dist/mapbox-gl.css';
import mapbox from 'mapbox-gl';
import * as turf from '@turf/turf'
import vueDanmaku from 'vue-danmaku'
import wordcloud from './compoment/wordcloud.vue'

let map;

export default {
    name: 'Dashboard',
    components: {
        vueDanmaku, wordcloud
    },
    computed: {},
    data() {
        return {
            danmus: [
                { content: ['我是弹幕', '好好好', '嘿嘿嘿', '哈哈哈'] },
                { content: ['111', '11', '1', '1'] },
                { content: ['我是弹幕', '好好好', '嘿嘿嘿', '哈哈哈'] },
                { content: ['我是弹幕', '好好好', '嘿嘿嘿', '哈哈哈'] },
                { content: ['我是弹幕', '好好好', '嘿嘿嘿', '哈哈哈'] },
                { content: ['我是弹幕', '好好好', '嘿嘿嘿', '哈哈哈'] },
                { content: ['我是弹幕', '好好好', '嘿嘿嘿', '哈哈哈'] },
                { content: ['我是弹幕', '好好好', '嘿嘿嘿', '哈哈哈'] },
            ],
            gridLayerShow: false,
            currentLayer: 'food',
            dataPOI: {}
        }
    },
    mounted() {
        const that = this;
        mapbox.accessToken = 'pk.eyJ1IjoiemhvbmdkaXNodW1hIiwiYSI6ImNsNXJoYXR5eTI2bGgzZW53d2didWF1c3AifQ.6vOplM2NQc_xnJW3aA5ZBA';
        map = new mapbox.Map({
            container: 'map-container', // container ID
            style: 'mapbox://styles/mapbox/streets-v12', // style URL
            center: [114.35400105, 30.6512685], // starting position [lng, lat]
            zoom: 10, // starting zoom
            minZoom: 10, // 最小缩放层级
        });
        // disable map rotation using right click + drag
        map.dragRotate.disable();

        // disable map rotation using touch rotation gesture  
        map.touchZoomRotate.disableRotation();

        (async () => {
            try {
                that.dataPOI = await that.createPoi(that.currentLayer)
                console.log(that.dataPOI); // 这里是获取到的数据
            } catch (error) {
                console.error('Error loading data:', error);
            }
        })();

        map.on('load', function () {
            viewportChanged()
            map.addSource('wuhanSource', {
                type: 'geojson',
                data: that.dataPOI
            });
            // 添加一个符号图层来显示文本标签
            map.addLayer({
                id: 'icons',
                type: 'symbol',
                source: 'wuhanSource',
                layout: {
                    'icon-image': 'restaurant',
                    'icon-size': 1.5, // 图标的大小，可以调整
                    'icon-allow-overlap': true, // 允许图标重叠

                    // 这里设置文本标签的内容，`get`表达式用于从每个要素的属性中获取评论数
                    'text-field': ['get', 'name'],
                    // 设置文本大小
                    'text-size': 12,
                    // 设置标签与图标的偏移量，避免遮挡
                    'text-offset': [0, 2],
                    // 文本字体
                    'text-font': ['Open Sans Bold', 'Arial Unicode MS Bold'],
                    // 最大宽度，以避免标签过长
                    'text-max-width': 8
                },
                paint: {
                    // 文本颜色
                    'text-color': '#000000',
                    // 文本的轮廓颜色和宽度，可以增加可读性
                    'text-halo-color': '#ffffff',
                    'text-halo-width': 1
                }
            });
        });
        map.on('zoomend', function () {
            console.log("zoomend");
            viewportChanged();
        })
        map.on('moveend', function () {
            console.log("moveend");
            viewportChanged();
        })
        //视口改变时调用此方法
        function viewportChanged() {
            that.stopDanmu();
            that.playDanmu();
            if (map.getSource('gridsource')) {
                map.removeLayer('grid-layer')
                map.removeSource('gridsource')
            } else {
                const squareGrid = createGrid(8);
                const randomPoints = that.dataPOI;
                for (let i = 0; i < squareGrid.features.length; i++) {
                    let gridCell = squareGrid.features[i];
                    // 将随机点分配到网格中
                    const pointsInCell = randomPoints.features.filter(point => turf.booleanPointInPolygon(point, gridCell.geometry));
                    that.danmus[i].content = pointsInCell.map(point => point.properties)

                    let points = that.danmus[i].content;
                    points.forEach(point => {
                        let temp = []
                        if (point.overall_rating) {
                            temp.push({ text: '评分:' + point.overall_rating, weight: 10 })
                        }
                        if (point.label) {
                            point.label.split(',').forEach(label => {
                                temp.push({ text: label, weight: 9 })
                            })
                        }
                        if (point.price) {
                            temp.push({ text: '价格:' + point.price, weight: 9 })
                        }
                        if (point.keywords) {
                            point.keywords.forEach((keyword) => {
                                let weight = Math.ceil(keyword[1] * 10)
                                if (weight < 1) {
                                    weight = 1
                                }
                                if (weight > 10) {
                                    weight = 10
                                }
                                temp.push({ text: keyword[0], weight: weight })
                            })
                        }
                        // 如果数组长度小于10，补充默认元素
                        while (temp.length < 10) {
                            temp.push({ text: '', weight: 1 });
                        }

                        // 随机排序数组
                        temp.sort(() => Math.random() - 0.5);

                        // 如果数组长度大于10，选择前10个元素

                        point.tagList = temp.slice(0, 10);
                    });

                    gridCell.properties.pointCount = parseInt(pointsInCell.length);
                }
                // 找出最大和最小点数
                const maxPointCount = Math.max(...squareGrid.features.map(cell => cell.properties.pointCount));
                const minPointCount = Math.min(...squareGrid.features.map(cell => cell.properties.pointCount));

                map.addSource('gridsource', {
                    type: 'geojson',
                    data: squareGrid // 这里假设squareGrid是已经生成的网格数据
                })

                map.addLayer({
                    id: 'grid-layer',
                    type: 'fill',
                    source: 'gridsource',
                    paint: {
                        'fill-color': [
                            'interpolate',
                            ['linear'],
                            ['get', 'pointCount'], // 假设每个格网包含点的数量在属性中称为 pointCount
                            minPointCount, '#008fbf', // 最小点数时的颜色，绿色
                            maxPointCount, '#a00000' // 最大点数时的颜色，红色
                        ],
                        'fill-opacity': 0.3 // 填充透明度
                    }
                });

                // Toggle layer visibility by changing the layout object's visibility property
                if (that.gridLayerShow) {
                    map.setLayoutProperty('grid-layer', 'visibility', 'visible');
                } else {
                    map.setLayoutProperty('grid-layer', 'visibility', 'none');
                }
            }
        }
        //根据视口生成格网
        function createGrid(num) {
            const result = getBounds(); //获取视口边界
            const { lngMin, lngMax, latMin, latMax } = result
            const bbox = [lngMin, latMin, lngMax, latMax];  //格网坐标 minX,minY,maxX,maxY. modify here.

            // 计算十个小矩形的边界坐标
            const bboxHeight = (bbox[3] - bbox[1]) / num; // 计算每个小矩形的高度
            const grids = [];
            for (let i = 0; i < num; i++) {
                const minY = bbox[1] + i * bboxHeight;
                const maxY = minY + bboxHeight;
                const gridBbox = [bbox[0], minY, bbox[2], maxY]; // 使用整个地图范围的宽度
                const gridRectangle = turf.bboxPolygon(gridBbox);
                grids.push(gridRectangle);
            }
            // 将十个小矩形组合成一个GeoJSON对象
            const squareGrid = turf.featureCollection(grids.reverse());

            return squareGrid;
        }

        // 获取视口边界
        function getBounds() {
            const bounds = map.getBounds()
            const result = {
                lngMax: bounds._ne.wrap().lng, // 最大经度
                lngMin: bounds._sw.wrap().lng, // 最小经度
                latMax: bounds._ne.wrap().lat, // 最大纬度
                latMin: bounds._sw.wrap().lat, // 最小纬度
            }
            return result
        }
    },
    methods: {
        hideDanmu() {
            this.$refs['danmaku'].forEach(item => item.hide())
        },
        showDanmu() {
            this.$refs['danmaku'].forEach(item => item.show());
        },
        pauseDanmu() {
            this.$refs['danmaku'].forEach(item => item.pause());
        },
        stopDanmu() {
            this.$refs['danmaku'].forEach(item => item.stop());
        },
        playDanmu() {
            this.$refs['danmaku'].forEach(item => item.play());
        },
        toggleGridLayer(e) {
            if (e) {
                map.setLayoutProperty('grid-layer', 'visibility', 'visible');
            } else {
                map.setLayoutProperty('grid-layer', 'visibility', 'none');
            }
        },
        selectPOILayer(e) {
            this.stopDanmu();

            let imageType;
            switch (e) {
                case 'scene':
                    imageType = 'attraction'
                    break;
                case 'food':
                    imageType = 'restaurant'
                    break;
                case 'fun':
                    imageType = 'theatre'
                    break;
                default:
                    imageType = 'marker'
                    break;
            }
            (async () => {
                try {
                    this.dataPOI = await this.createPoi(e)
                    map.getSource('wuhanSource').setData(this.dataPOI);
                    map.setLayoutProperty('icons', 'icon-image', imageType);
                    console.log(this.dataPOI); // 这里是获取到的数据
                } catch (error) {
                    console.error('Error loading data:', error);
                }
            })();
        },
        createPoi(type) {
            //生成poi点
            if (type != 'food' && type != 'fun' && type != 'scene') {
                return;
            }
            return new Promise((resolve, reject) => {
                // 使用 Fetch API 加载 GeoJSON 文件
                fetch(`./data/${type}.json`)
                    .then(response => response.json()) // 将响应转换为 JSON 格式
                    .then(data => {
                        // 使用 Turf.js 处理加载的 GeoJSON 数据
                        const turfData = turf.featureCollection(data.features);
                        resolve(turfData);
                    })
                    .catch(error => {
                        reject(error);
                    });
            })
        }
    },

}
</script>

<style lang="scss" scoped>
#map-container {
    padding: 0;
    margin: 0;
    width: 100vw;
    height: calc(100vh - 50px);
    overflow: hidden;
}

.danmaku {
    position: absolute;
    left: 0;
    top: 0;
    width: 100vw;
    height: calc((100vh - 50px)/8);
    z-index: 99;
    pointer-events: none;
    /* 禁止弹幕元素被选中 */
    /* background-color: #fff; */
}

.danmaku-name {
    height: calc((100vh - 50px)/8);
}

.danmu-control-pannel {
    position: absolute;
    left: 10px;
    top: 10px;
    z-index: 999;
}

.layer-control-pannel {
    position: absolute;
    right: 10px;
    top: 10px;
    z-index: 999;
}
</style>
