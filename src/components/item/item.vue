<template>
    <div class="item">#{{item.id}}-{{item.value}}</div>
</template>

<script>
export default {
    props:{
        item: Object
    }
}
</script>

<style scoped>
    .item{
        /* height: 40px; */
        border: 1px solid #000;
        padding: 20px 0;
    }
</style>