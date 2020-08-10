<template>
  <div ref="preview" class="vue-run-sfc-preview">
    <vue-element-loading
      :active="loading"
      spinner="spinner"
      :color="themeColor"
    />

    <iframe
      sandbox="allow-modals allow-forms allow-pointer-lock allow-popups allow-same-origin allow-scripts"
      scrolling="yes"
      ref="iframe"
      frameborder="0"
      style="width: 100%;height: 100%;border: none;"
    ></iframe>
  </div>
</template>

<script>
// 参考: https://github.com/QingWei-Li/vuep.run/blob/master/src/components/preview.vue
import VueElementLoading from 'vue-element-loading'
const { debounce } = require('throttle-debounce')

export default {
  components: {
    VueElementLoading
  },
  props: {
    jsLabs: {
      type: Array,
      default: () => []
    },
    js: {
      type: [Array, String],
      default: () => []
    },
    cssLabs: {
      type: Array,
      default: () => []
    },
    css: {
      type: [Array, String],
      default: () => []
    },
    value: {
      type: Object,
      required: true
    },
    themeColor: {
      type: String,
      default: '#409eff'
    }
  },
  mounted () {
    // this.initdoc()
    this.setHTML()
  },
  data () {
    return {
      iframe: null,
      iframeDocument: null,
      loading: true
    }
  },
  watch: {
    value () {
      this.setHTML()
    }
  },
  methods: {
    // 根据内容更改高度
    changeHeight () {
      if (!this.debounceChangeHeight) {
        this.debounceChangeHeight = debounce(300, () => {
          const iframe = this.iframe
          const iframeDocument = this.iframeDocument
          iframe.style.display = 'block'
          const extendHeight = 10 // 额外的高度(避免出现滚动条)
          console.log(iframeDocument.documentElement)
          const height =
            iframeDocument.documentElement.clientHeight + extendHeight
          iframe.style.height = height + 'px'
          if (this.$refs.preview) {
            this.$emit('change-height', this.$refs.preview.clientHeight)
          }
        })
      }
      this.debounceChangeHeight()
    },
    createHtmltmp () {
      var html = ''
      console.log('this.value', this.value)
      let { styles = [], script = '', template, errors, iscommon } = this.value
      if (this.value.iscommon === 1) {
        html = `
         ${template}`
      } else if (this.value.iscommon === 0) {
        var stylesTags = this.cssLabs.map(
          style => `<link rel="stylesheet" href="${style}" />`
        )
        var scriptTags = this.jsLabs.map(
          script => `<script src="${script}"><\/script>`
        )
        var js = Array.isArray(this.js) ? this.js : [this.js]
        var css = Array.isArray(this.css) ? this.css : [this.css]
        var script2 = this.getScript(script, template)
        html = `
         <!DOCTYPE html>
           <html>
             <head>
               ${stylesTags.join('\n')}
               <style>${css.join('\n')}</style>

               <script src='https://cdn.jsdelivr.net/npm/vue/dist/vue.js'><\/script>
               ${scriptTags.join('\n')}
               <script>${js.join('\n')}<\/script>
               <script>
                 // 错误处理
                 var errorHandler = function(error) {
                   var el = document.getElementById('error')
                   el.innerHTML = '<pre style="color: red">' + error.stack +'</pre>'
                 }
                 Vue.config.warnHandler = function(msg) { errorHandler(new Error(msg)) }
                 Vue.config.errorHandler = errorHandler
                 Vue.config.productionTip = false
                 Vue.config.devtools = false
               <\/script>
             </head>
             <body id="body">
               <div><pre id="error" style="color: red"></pre></div>
               <div id="box">
                  <style>${styles.join('\n')}</style>
                  <div id="app">
                  </div>
                  <script>
                  ${script2}
                  <\/script>
               </div>
             </body>
         </html>`
      }
      return html
    },
    getScript (script, template) {
      return ` try {
          var exports = {};
          ${script}
          var component = exports.default;
          // 如果定义了 template函数, 则无需 template
          component.template = component.template || ${template}
        } catch (error){
          errorHandler(error)
        }

        new Vue(component).$mount('#app')
      `
    },
    // 设置html
    setHTML () {
      console.log('setHTML iframe :')
      const iframe = this.$refs.iframe
      const iframeDocument = iframe.contentWindow.document
      var html = this.createHtmltmp()
      window.gethtml = `${html}`
      iframe.src = 'javascript:parent.gethtml;'

      iframe.onload = () => {
        console.log('iframe onload:', iframeDocument.body.offsetHeight)
        this.loading = false
        // this.iframe = iframe
        // this.iframeDocument = iframeDocument
        // if (!this.height) {
        //   this.$nextTick(() => {
        //     this.changeHeight()
        //     iframe.contentWindow.addEventListener(
        //       'resize',
        //       this.changeHeight,
        //       false
        //     )
        //   })
        // }
      }

      iframe.error = () => {
        this.loading = false
      }
    }
  },
  beforeDestory () {
    if (
      this.iframe &&
      this.iframe.contentWindow &&
      this.iframe.contentWindow.addEventListener
    ) {
      this.iframe.contentWindow.removeEventListener('resize', this.changeHeight)
    }
  }
}
</script>

<style>
/* 展示区样式 */
.vue-run-sfc-preview {
  background: white;
  padding: 20px 15px;
  height: 100%;
}
</style>
