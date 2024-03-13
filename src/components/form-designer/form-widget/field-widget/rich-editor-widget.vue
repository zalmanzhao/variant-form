<template>
  <form-item-wrapper :designer="designer" :field="field" :rules="rules" :design-state="designState"
                     :parent-widget="parentWidget" :parent-list="parentList" :index-of-parent-list="indexOfParentList"
                     :sub-form-row-index="subFormRowIndex" :sub-form-col-index="subFormColIndex" :sub-form-row-id="subFormRowId">
    <template v-if="previewState">
      <div v-html="fieldModel" />
    </template>
    <template v-else>
      <div style="border: 1px solid #ccc; margin-top: 10px;">
        <!-- 工具栏 -->
        <Toolbar
            style="border-bottom: 1px solid #ccc"
            :editor="editor"
            :defaultConfig="toolbarConfig"
            :mode="editorConfig.mode"
        />
        <Editor ref="fieldEditor" :style="{'height': field.options.height + 'px', 'overflow-y': 'hidden'}" v-model="fieldModel"
                    :defaultConfig="editorConfig" @onCreated="onCreated"
                    @onChange="handleRichEditorChangeEvent" :mode="editorConfig.mode">
        </Editor>
      </div>
    </template>
  </form-item-wrapper>
</template>

<script>
  import FormItemWrapper from './form-item-wrapper'
  import { Editor, Toolbar } from '@wangeditor/editor-for-vue'
  import '@wangeditor/editor/dist/css/style.css'
  import { Boot } from '@wangeditor/editor'
  import markdownModule from '@wangeditor/plugin-md'
  Boot.registerModule(markdownModule)
  import emitter from '@/utils/emitter'
  import i18n from "@/utils/i18n"
  import fieldMixin from "@/components/form-designer/form-widget/field-widget/fieldMixin"

  export default {
    name: "rich-editor-widget",
    componentName: 'FieldWidget',  //必须固定为FieldWidget，用于接收父级组件的broadcast事件
    mixins: [emitter, fieldMixin, i18n],
    props: {
      field: Object,
      parentWidget: Object,
      parentList: Array,
      indexOfParentList: Number,
      designer: Object,

      designState: {
        type: Boolean,
        default: false
      },

      subFormRowIndex: { /* 子表单组件行索引，从0开始计数 */
        type: Number,
        default: -1
      },
      subFormColIndex: { /* 子表单组件列索引，从0开始计数 */
        type: Number,
        default: -1
      },
      subFormRowId: { /* 子表单组件行Id，唯一id且不可变 */
        type: String,
        default: ''
      },

    },
    components: {
      FormItemWrapper,
      Editor,
      Toolbar
    },
    inject: ['refList', 'formConfig', 'globalOptionData', 'globalModel'],
    data() {
      return {
        editor: null,
        oldFieldValue: null, //field组件change之前的值
        fieldModel: null,
        rules: [],
        toolbarConfig: { },
        editorConfig: { mode: 'simple', MENU_CONF: {} },
        valueChangedFlag: false, //vue2-editor数据值是否改变标志
      }
    },
    computed: {
      realUploadURL() {
        let uploadURL = this.field.options.uploadURL
        if (!!uploadURL && ((uploadURL.indexOf('DSV.') > -1) || (uploadURL.indexOf('DSV[') > -1))) {
          let DSV = this.getGlobalDsv()
          console.log('test DSV: ', DSV)  //防止DSV被打包工具优化！！！
          return evalFn(this.field.options.uploadURL, DSV)
        }

        return this.field.options.uploadURL
      },
    },
    beforeCreate() {
      /* 这里不能访问方法和属性！！ */
    },

    created() {
      /* 注意：子组件mounted在父组件created之后、父组件mounted之前触发，故子组件mounted需要用到的prop
         需要在父组件created中初始化！！ */
      this.initFieldModel()
      this.registerToRefList()
      this.initEventHandler()
      this.buildFieldRules()

      this.handleOnCreated()

      this.editorConfig.MENU_CONF['uploadImage'] =  {
        fieldName: 'file',
        server: this.realUploadURL,
        maxFileSize: this.field.options.fileMaxSize * 1024 * 1024, // 
        // 最多可上传几个文件，默认为 100
        maxNumberOfFiles: this.field.options.limit,

        base64LimitSize: 100 * 1024, // insert base64 format, if file's size less than 100kb
        // 跨域是否传递 cookie ，默认为 false
        withCredentials: this.field.options.withCredentials,
        timeout: 10 * 1000, // 10 秒
      }
    },

    mounted() {
      this.handleOnMounted()
    },

    beforeDestroy() {
      this.unregisterFromRefList()
      const editor = this.editor
      if (editor == null) return
      editor.destroy() // 组件销毁时，及时销毁 editor ，重要！！！
    },

    methods: {
      onCreated(editor) {
        this.editor = Object.seal(editor); // 【注意】一定要用 Object.seal() 否则会报错
      },

      handleRichEditorChangeEvent(editor) {
        this.valueChangedFlag = true
        this.syncUpdateFormModel(editor.getHtml())
      },
    }
  }
</script>

<style lang="scss" scoped>
  @import "../../../../styles/global.scss"; //* form-item-wrapper已引入，还需要重复引入吗？ *//

  .full-width-input {
    width: 100% !important;
  }

</style>