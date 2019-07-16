<template>
  <section class='home'>
    <button @click="changeTheme">加载css</button>
    <h1>This is home page</h1>
  </section>
</template>

<script>
export default {
  name: 'home',
  mixins: [],
  components: {},
  props: {},
  computed: {},
  watch: {},
  data () {
    return {
      theme: ''
    }
  },
  created () {},
  mounted () {
    this.changeTheme()
  },
  methods: {
    changeTheme () {
      this.theme = this.theme === 'default' ? 'red' : 'default'
      const body = document.body
      body.className = ''
      this.theme !== 'default' && body.classList.add(this.theme)
      import(
        /* webpackChunkName: "theme-[request]" */
        /* webpackInclude: /\.js$/ */
        `./${this.theme}`)
    }
  },
  beforeDestroy () {}
}
</script>

<style lang='less' scoped>
.home {
  // color: rgb(217, 231, 14);
}
</style>
