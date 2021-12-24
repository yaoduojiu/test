<template>
  <a-card :bordered="false">
    <a-tabs defaultActiveKey="1">
      <a-tab-pane tab="在库明细" key="1" >
        <detail-list></detail-list>
      </a-tab-pane>
      <a-tab-pane tab="按物料查询" key="2" >
        <material-list></material-list>
      </a-tab-pane>
      <a-tab-pane tab="按库位查询" key="3" >
        <location-list></location-list>
      </a-tab-pane>
    </a-tabs>
    <!-- table区域-begin -->
   
  </a-card>
</template>

<script>

  import detailList from './modules/detailList'
  import materialList from './modules/materialList'
  import locationList from './modules/locationList'
  export default {
    name: 'inStockSearchList',
    components: {
      detailList,
      materialList,
      locationList
    }
  }
</script>