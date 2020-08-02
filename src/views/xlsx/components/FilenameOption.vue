<template>
  <div style="display:inline-block;">
    <!-- <label class="radio-label" style="padding-left:0;">文件名: </label> -->
    <el-input v-model="filename" :placeholder="placeholder" style="width:271px;" prefix-icon="el-icon-document" />
  </div>
</template>

<script>
export default {
  props: {
    value: {
      type: String,
      default: ''
    },
    username: {
      type: String,
      default: '明玉钗'
    }
  },
  computed: {
    filename: {
      get() {
        return this.value
      },
      set(val) {
        this.$emit('input', val)
      }
    },
    placeholder() {
      return '文件名 (默认：' + (this.username ? this.username : '明玉钗') + '的7月考勤表)'
    }
  }
}
</script>
