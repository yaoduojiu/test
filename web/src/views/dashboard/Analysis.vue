<template>
  <div id="container" style="width: 100%;">
    <!-- <index-chart v-if="indexStyle==1"></index-chart>
    <index-bdc v-if="indexStyle==2"></index-bdc>
    <index-task v-if="indexStyle==3"></index-task>
    <div style="width: 100%;text-align: right;margin-top: 20px">
      请选择首页样式：
      <a-radio-group v-model="indexStyle">
        <a-radio :value="1">统计图表</a-radio>
        <a-radio :value="2">统计图表2</a-radio>
        <a-radio :value="3">任务表格</a-radio>
      </a-radio-group>
    </div> -->
    <div class="flex flex-jcc flex-aic" style="width: 100%;height: 100%;">
      <span style="font-size: 38px;color: #308bd8;">
        欢迎回来，一码追溯系统
      </span>
    </div>
  </div>
</template>

<script>
  import IndexChart from './IndexChart'
  import IndexTask from "./IndexTask"
  import IndexBdc from './IndexBdc'

  export default {
    name: "Analysis",
    components: {
      IndexChart,
      IndexTask,
      IndexBdc
    },
    data() {
      return {
        indexStyle:1
      }
    },
    created() {

    },
    mounted() {
      setTimeout(() => {
        document.getElementById('container').style.height = document.body.offsetHeight - 200 + 'px'
      })
    },
    methods: {

    }
  }
</script>
<style scoped>
.flex {
  display: flex;
}
.flex-jcc {
  justify-content: center;
}
.flex-aic {
  align-items: center;
}
</style>