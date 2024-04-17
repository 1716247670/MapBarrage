<template>
    <div class="word-container">
        <div class="grid-item" v-for="(word, index) in fullList" :key="index" :class="{ 'main-word': word.main }"
            :style="{ fontSize: fontSize(word.weight), color: color(word.weight) }">
           <span v-if="word.text"> {{ word.text }}</span>
        </div>
    </div>
</template>

<script>

export default {
    name: 'WordCloud',
    props: {
        mainWord: {
            type: Object,
            default: () => ({ text: '无', weight: 10 }),
        },
        keywordsList: {
            type: Array,
            default: () => []
        }
    },
    data() {
        return {
            // 颜色数组，从权重最高到最低
            colors: ["#008fbf", "#239fcf", "#47b0df", "#6ac0ef", "#8dd0ff", "#dae695", "#ffb8b8", "#df7b7b", "#c03d3d", "#a00000"]
        };
    },
    mounted() {

    },
    methods: {
        fontSize(weight) {
            // 基础字体大小加上权重乘以一个因子
            return `${14 + weight * 1}px`;  // 每个权重单位增加2px
        },
        color(weight) {
            return this.colors[weight - 1];  // 由于权重从1开始，数组索引从0开始，所以需要减1
        }
    },
    computed: {
        fullList() {
            let list = [...this.keywordsList];
            list.splice(5, 0, { ...this.mainWord, main: true });
            return list;
        }
    }
};
</script>

<style scoped>
.word-container {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    /* 定义四列 */
    grid-template-rows: repeat(3, 1fr);
    /* 定义三行 */
    gap: 5px;
    /* 网格之间的间隙 */
    width: 100%;
    height: 100%;
    align-items: center;
    justify-items: center;
}

.grid-item.main-word {
    grid-column: span 2;
    font-weight: bold;
    color: #333;
    background-color: rgba(255,255,255,0.3);
    font-family: 'SimHei', 'Microsoft YaHei', sans-serif; /* 这里选择了常用的中文字体，也可以选择其他 */
    text-shadow: 2px 2px 4px rgba(255,255,255,0.5); /* 添加阴影效果，可调整颜色和大小 */
}

.grid-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    overflow: hidden;
    text-align: center;
    text-overflow: ellipsis;
    white-space: nowrap;
    background-color: rgba(255,255,255,0.8);
    border-radius: 5px;
}
</style>