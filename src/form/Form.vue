<template>
  <form>
    <slot></slot>
  </form>
</template>

<script>
export default {
  name: 'WangForm',
  provide () {
    return {
      form: this
    }
  },
  props: {
    model: {
      type: Object
    },
    rules: {
      type: Object
    }
  },
  methods: {
    validate (cb) {
      // 过滤出含有prop验证的节点 执行validate方法 返回promise对象
      const tasks = this.$children
        .filter(child => child.prop)
        .map(child => child.validate())

      Promise.all(tasks)
        .then(() => cb(true))
        .catch(() => cb(false))
    }
  }
}

</script>
<style>
</style>