<template>
  <a-card :bordered="false">
     <a-tabs defaultActiveKey="1">
      <a-tab-pane tab="原材料" key="1" >
        <MaterialList></MaterialList>
      </a-tab-pane>
      <a-tab-pane tab="产成品" key="2" >
        <ProductList></ProductList>
      </a-tab-pane>
    </a-tabs>
  </a-card>
</template>
<script>
  import MaterialList from './materialList'
  import ProductList from './productList'
  export default {
    name: "traceList",
    components: {
      MaterialList,
      ProductList
    },
  }
</script>