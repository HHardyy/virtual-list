<template>
  <div id="app">
    <!-- 告诉每一项的高度，每个可视区有几项, 数据源 -->
   <virtualList :size='40' :remain='8' :items='items'>
     <item slot-scope="{item}" :item="item"></item>
   </virtualList>
  </div>
</template>

<script>
import VirtualList from '@/components/vurtual-list'
import Item from '@/components/item/item'

// MOCK DATA
let items = []
for(let i=0; i< 10000; i++) {
  items.push({
    id:i,
    value:`#item ${i}`
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
