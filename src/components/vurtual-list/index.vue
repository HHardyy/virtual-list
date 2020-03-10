<template>
    <div class="viewport" ref="viewport" @scroll="handleScroll">
        <!-- 滚动条 -->
        <div class="scroll-bar" ref="scrollBar"></div>

        <!-- 渲染列表 -->
        <div class="scroll-list" :style="{transform:`translateY(${offset})`}">
            <div v-for="item in visibleData" :vid="item.id" :key="item.id">
                 <slot :item="item"></slot>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    props:{
        size: Number,  // 每个高度
        remain: Number,   // 可视区数量
        items: Array  // 列表数据
    },
    data(){
        return {
            start: 0,
            end: this.remain,
            offset: 0
        }
    },
    computed:{
        prevCount(){   // 为避免单屏幕上下无连接，前面渲染一屏
            return Math.min(this.start, this.remain)
        },
        nextCount(){   // 为避免单屏幕上下无连接，后面渲染一屏
            return Math.min(this.remain, this.items.length - this.end)
        },
        visibleData() {
            let start = this.start - this.prevCount
            let end = this.end + this.nextCount
            return this.items.slice(start, end)
        }
    },
    mounted(){
        this.init()
    },
    methods:{
        handleScroll(){
            // 滚动了多高
            let scrollTop = this.$refs.viewport.scrollTop

            // 当前应该从第几个开始渲染
            this.start = Math.floor(scrollTop / this.size)   //高度除以每个高度取整
            this.end = this.start + this.remain;
            
            //偏移量，让列表始终出现在可视区
            this.offset = this.start * this.size - this.size * this.prevCount +'px'
        },
        init(){
            this.$refs.viewport.style.height = this.size * this.remain + 'px'
            this.$refs.scrollBar.style.height = this.items.length * this.size +'px'
        }
    }
}
</script>

<style scoped>
    .viewport{
        overflow-y:scroll;
        position: relative;
        box-shadow: 0 0 10px orange;
    }
    .scroll-list{
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
    }
</style>