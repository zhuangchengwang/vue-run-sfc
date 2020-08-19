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
    token: String,
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
    codecompileurl: {
        type: String,
        default: '#'
    },
    themeColor: {
      type: String,
      default: '#409eff'
    }
  },
  mounted () {
    // this.initdoc()
    // console.log("mounted:")
    this.setHTML()
  },
  data () {
    return {
      iframe: null,
      iframeDocument: null,
      loading: false
    }
  },
  watch: {
    value () {
      // console.log("value change:")
      this.setHTML()
    }
  },
  methods: {
    changeloading(loading=true){
      this.loading = loading;
    },
    // 根据内容更改高度
    changeHeight () {
      if (!this.debounceChangeHeight) {
        this.debounceChangeHeight = debounce(300, () => {
          const iframe = this.iframe
          const iframeDocument = this.iframeDocument
          iframe.style.display = 'block'
          const extendHeight = 10 // 额外的高度(避免出现滚动条)

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
      return new Promise((resolve,reject)=>{
        let htmls = ['html','css','javascript','vue'];
        if(htmls.indexOf(this.value.language)>-1){
          var html = ''
          // console.log('this.value', this.value)
          let { styles = [], script = '', template, errors, iscommon,language } = this.value
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
          resolve(html)
        }else{
          //ajax 请求
          // console.log(this.value);
          if(!this.value.template){
           resolve("代码改变,就可以在此实时预览效果~~~")
           return;
          }

          let url = this.codecompileurl;
          let axios = window.axios.create({
          timeout: 5000, // request timeout  设置请求超时时间
          responseType: "json",
          // withCredentials: true, // 是否允许带cookie这些
          headers: {
            "Content-Type": "application/json;charset=utf-8"
          }
      });
          let data = new FormData();
          data.append('code',this.value.template);
          data.append('language',this.value.language);
          data.append('_token',this.token);
          axios.post(this.codecompileurl,data)
                .then((response) => {
                  //console.log(response)
                   resolve('<pre>'+response.data.data+'</pre>')
                })
                .catch(function (error) { // 请求失败处理
                  //console.log(error);
                });

        }
      })

    },
    getScript (script, template) {
      return ` try {
          var exports = {};
          "use strict";
          Object.defineProperty(exports, "__esModule", {
            value: true
          });
          exports.default = void 0;
          ${script}
          exports.default = _default;
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

      const iframe = this.$refs.iframe
      const iframeDocument = iframe.contentWindow.document
      let pro = this.createHtmltmp()
      pro.then((html)=>{

        window.gethtml = `${html}`
        iframe.src = 'javascript:parent.gethtml;'

        iframe.onload = () => {
          //console.log('iframe onload:')
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
      })
      setTimeout(()=>{
         this.loading = false
      },3500)

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
