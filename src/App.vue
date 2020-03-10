<template>
  <div id="app">
    <!-- 告诉每一项的高度，每个可视区有几项, 数据源 -->
   <virtualList :size='100' :remain='8' :items='items' variable>
     <item slot-scope="{item}" :item="item"></item>
   </virtualList>
  </div>
</template>

<script>
import VirtualList from '@/components/vurtual-list'
import Item from '@/components/item/item'
// 使用mock数据来模拟未确定的数据以及不定宽高的盒子
import Mock from 'mockjs'

// MOCK DATA
let items = []
for(let i=0; i< 10000; i++) {
  items.push({
    id:i,
    value: Mock.Random.sentence(5,20)
  })
}

export default {
  name: 'App',
  data(){
    return {
      items
    }
  },
  components: {
    VirtualList,
    Item
  },
  methods: {
    functiona()   {}
  }
}
</script>


<style scoped>
  *{
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }
</style>
