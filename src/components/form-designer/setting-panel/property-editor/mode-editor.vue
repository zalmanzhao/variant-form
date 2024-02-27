<template>
  <el-form-item :label="i18nt('designer.setting.mode')">
    <el-select v-model="optionModel.mode">
      <el-option v-for="item in modes" :key="item.value" :label="item.label"
                 :value="item.value">
      </el-option>
    </el-select>
  </el-form-item>
</template>

<script>
  import i18n from "@/utils/i18n"

  export default {
    name: "mode-editor",
    mixins: [i18n],
    props: {
      designer: Object,
      selectedWidget: Object,
      optionModel: Object,
    },
    data() {
      return {
        modes: [
          {label: 'MySQL', value: 'mysql'},
          {label: 'JavaScript', value: 'javascript'},
          {label: 'JSON', value: 'json'},
          {label: 'CSS', value: 'css'},
          {label: 'SQL', value: 'sql'},
        ],
      }
    }
  }
</script>

<style scoped>

</style>
