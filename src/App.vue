<template>
  <div id="app">
    <SlideVerify
      ref="slideDiv"
      :w="400"
      :h="200"
      @success="handleSuccess"
      @fail="handleError"
    />
  </div>
</template>

<script>
import SlideVerify from './components/slide-verify'
export default {
  name: 'App',
  components: { SlideVerify },
  data () {
    return {}
  },
  methods: {
    handleSuccess () {
      alert('校验通过')
    },
    handleError () {
      alert('校验不通过')
    }
  }
}
</script>

<style lang="less">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
