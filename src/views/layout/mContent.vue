<template>
    <div class="m-content" :class="$store.state.show_login?'m-hidden':''">
      <keep-alive>
        <router-view ></router-view>
      </keep-alive>
    </div>

</template>

<script type="text/ecmascript-6">
    export default {
        data() {
            return {
                name: ''
            }
        },
        components: {},
        methods: {},
        created() {

        }
    }
</script>
<style lang="less" rel="stylesheet/less" scoped>
.m-content{
  /*padding: 10px 0;*/
  margin-bottom: 100px;
}
.m-hidden{
    height: 92vh;
    overflow: hidden;
}
</style>
