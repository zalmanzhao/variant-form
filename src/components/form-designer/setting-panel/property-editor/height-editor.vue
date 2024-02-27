<template>
  <el-form-item :label="i18nt('designer.setting.height')">
    <el-input type="number" @input.native="inputNumberHandler" class="hide-spin-button"
              min="10" v-model="height" style="width: 100%"></el-input>
  </el-form-item>
</template>

<script>
  import i18n from "@/utils/i18n"
  import propertyMixin from "@/components/form-designer/setting-panel/property-editor/propertyMixin"

  export default {
    name: "height-editor",
    mixins: [i18n, propertyMixin],
    props: {
      designer: Object,
      selectedWidget: Object,
      optionModel: Object,
    },
    computed: {
      height: {
        get() {
          return this.optionModel['height']
        },

        set(newValue) {
          if (!newValue || isNaN(newValue)) {
            this.optionModel.height = null
          } else {
            this.optionModel.height = Number(newValue)
          }
        }
      },

    }
  }
</script>

<style scoped>

</style>
