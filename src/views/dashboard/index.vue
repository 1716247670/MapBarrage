<template>
    <div>
        <!-- <vue-danmaku class="danmaku" ref="danmakuRef" v-model:danmus="danmus" useSlot loop :speeds="100" :channels="10">
            <template v-slot:dm="{ danmu }">
                <div class="danmaku-name">{{ danmu }}</div>
            </template>
</vue-danmaku> -->
        <div id="map-container">
        </div>
    </div>
</template>

<script>
import 'mapbox-gl/dist/mapbox-gl.css';
import mapbox from 'mapbox-gl';
import vueDanmaku from 'vue-danmaku'

export default {
    name: 'Dashboard',
    computed: {},
    data() {
        return {
        }
    },
    mounted() {
        console.log('mounted');
        mapbox.accessToken = 'pk.eyJ1IjoiemhvbmdkaXNodW1hIiwiYSI6ImNsNXJoYXR5eTI2bGgzZW53d2didWF1c3AifQ.6vOplM2NQc_xnJW3aA5ZBA';
        const map = new mapbox.Map({
            container: 'map-container', // container ID
            style: 'mapbox://styles/mapbox/streets-v12', // style URL
            center: [114.35400105, 30.6512685], // starting position [lng, lat]
            zoom: 10, // starting zoom
            minZoom: 10, // 最小缩放层级
        });
        console.log(map);
    },
    methods: {},

}
</script>

<style lang="scss" scoped>
#map-container {
    padding: 0;
    margin: 0;
    width: 100vw;
    height: 100vh;
    overflow: hidden;
}

.danmaku {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    z-index: 999;
    pointer-events: none;
    /* 禁止弹幕元素被选中 */
    /* background-color: #fff; */
}
</style>
