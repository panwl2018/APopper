<template>
  <div id="app">
    <template v-for="node in nodeList">
      <APopper  :key="node.key" :tips-target-text="node.name" :active-length="220">
       <span slot="tipsContent">{{node.name}}</span>
      </APopper>
    </template>
    
  </div>
</template>

<script>
import APopper from './components/APopper.vue'

export default {
  name: 'App',
  components: {
    APopper
  },
  data() {
    return {
      nodeList: [
        {key:'1',name:'撸起袖子加油干 决战决胜攻坚战'},
        {key:'2',name:'经济薄弱村扶持项目描述'},
        {key:'3',name:'天津港保税区职工再就业帮扶办法'},
        {key:'4',name:'关于加大结对帮扶工作力度的通知'},
        {key:'5',name:'学困生的帮扶方法探究'},
        {key:'6',name:'关于认真做好2017年计划生育“三结合”工'},
        {key:'7',name:'新时期贫困人口帮扶工作指引'},
        {key:'8',name:'关于进一步加强贫困户结对帮扶工作的通知'},
        {key:'9',name:'县“冬季行动”企业帮扶实施方案'},
        {key:'10',name:'关于建立精准扶贫“帮扶活动日”制度的'}
      ]
    };
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
  height: 100px;
  overflow-y: auto;
}
</style>
