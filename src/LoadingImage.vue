<template>
  <img :title=title />
</template>

<script>
export default {
    props: ['src'],
    data: function () { return (
      {
        start: Date.now(),
        ttfb: null,
        total: null,
      }
    )},
    mounted () {
      this.$el.addEventListener("loadstart", () => {
        this.ttfb = Date.now() - this.start
      })
      this.$el.addEventListener("load", () => {
        this.total = Date.now() - this.start
      })
      this.$el.src = this.src
    },
    computed: {
      title: function () {
        return (
          this.ttfb
            ? `ttfb ${this.ttfb} ms\n`
            : ''
          ) + `load: ${this.total} ms`
      }
    },
    watch: {
      src: function (newSrc) {
        // reset img for visual feedback
        this.$el.src = "data:image/gif;base64,R0lGODlhAQABAAD/ACwAAAAAAQABAAACADs%3D"
        setTimeout(() => {
          this.start = Date.now()
          this.$el.src = newSrc
        }, 0)
      }
    }
}
</script>
