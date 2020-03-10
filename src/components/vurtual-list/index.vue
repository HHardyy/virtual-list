<template>
    <div class="viewport" ref="viewport" @scroll="scrollFn">
        <!-- 滚动条 -->
        <div class="scroll-bar" ref="scrollBar"></div>

        <!-- 渲染列表 -->
        <div class="scroll-list" :style="{transform:`translateY(${offset})`}">
            <div v-for="item in visibleData" :vid="item.id" :key="item.id" ref="items">
                 <slot :item="item"></slot>
            </div>
        </div>
    </div>
</template>

<script>
import throttle from 'lodash/throttle'
export default {
    props:{
        size: Number,  // 每个高度
        remain: Number,   // 可视区数量
        items: Array,  // 列表数据
        variable: Boolean // 是否定宽
    },
    data(){
        return {
            start: 0,
            end: this.remain,
            offset: 0,
            positions: []
        }
    },
    created(){
        this.scrollFn = throttle(this.handleScroll, 200, {leading:false})
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
    updated(){
        // 页面渲染完成之后，根据当前展示的数据，更新缓冲区的内容
        this.$nextTick(()=>{
            let nodes = this.$refs.items
            if(!(nodes && nodes.length > 0)) {
                return
            }
            nodes.forEach(node => {
                let {height} = node.getBoundingClientRect()
                let id = node.getAttribute('vid') - 0
                let oldHeight = this.positions[id].height
                let val = oldHeight - height   // 计算当前高度与之前变化
                if(val) {  // 如果缓存和真实高度一样就不需要任何操作了
                    this.positions[id].height = height
                    this.positions[id].bottom = this.positions[id].bottom - val // 底部增加了
                    // 链表
                    for(let i = id + 1; i < this.positions.length; i++) {
                        this.positions[i].top = this.positions[i - 1].bottom
                        this.positions[i].bottom = this.positions[i].bottom - val
                    }
                }
            })
            // 只要更新过，就计算滚动条的最新高度
            this.$refs.scrollBar.style.height = this.positions[this.positions.length - 1].bottom
        })
    },
    mounted(){
        this.init()   // 初始化高度
        this.catchList()  // 如果加载完毕，缓存每一项的高度,top,bottom
    },
    methods:{
        // 获取每项高度，top，bottom, 在滚屏的时候再去获取渲染的dom的高度，top，bottom
        catchList(){
            this.positions = this.items.map((item, index) => ({
                top: index * this.size,
                height: this.size,
                bottom: (index + 1) * this.size
            }));
        },
        // 查找当前滚动的需要找到的值
        getStartIndex(value){  
            let start = 0
            let end = this.positions.length - 1
            let temp = null

            while(start<=end) {
                let middleIndex = parseInt((start+end)/2)
                let middleValue = this.positions[middleIndex].bottom   // 找到当前中间的人的结尾点
               
                if(middleValue == value) {  // 如果找到了，就直接返回当前的下一个人
                    return middleIndex + 1
                }else if(middleValue<value) {  // 在右边
                    start = middleIndex + 1
                }else if(middleValue>value) {  // 在左边
                    if(temp == null || temp > middleIndex){
                        temp = middleIndex
                    }
                    end = middleIndex - 1
                }
            }
            return temp
        },
        // 屏幕滚动
        handleScroll(){
            // 1、算出来当前滚过去多少个了，当前应该从第几个开始显示
            // 滚动了多高
            let scrollTop = this.$refs.viewport.scrollTop

            if(this.variable){  // 如果有variable，就说明使用二分查找
                this.start = this.getStartIndex(scrollTop)
                this.end = this.start + this.remain
                this.offset = this.positions[this.start - this.prevCount] ? this.positions[this.start - this.prevCount].top+'px' : 0
            } else {
                // 当前应该从第几个开始渲染
                // 2、获取当前应该从第几个开始渲染
                this.start = Math.floor(scrollTop / this.size)   //高度除以每个高度取整
                // 3、计算当前结尾的位置
                this.end = this.start + this.remain;
                
                //偏移量，让列表始终出现在可视区
                this.offset = this.start * this.size - this.size * this.prevCount +'px'
            }
        },
        // 初始化
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