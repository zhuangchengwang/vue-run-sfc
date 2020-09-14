<template>
  <div>
    <section >
      <vue-run-sfc
        :js-labs="jsLabs"
        :css-labs="cssLabs"
        title="测试DEMO"
        :open="false"
        :code="code"

        codelanguage="html"
        codecompileurl="http://wiki.xi13n.cn/runcode/run"
        @expandchange="expandChange"
      />
      <!-- <h2>DEMO 源码</h2>
      <codemirror
        style="border-top: 1px solid #eee"
        :options="codemirrorOption"
        v-model="exampleCode"
      /> -->
    </section>
  </div>
</template>

<script>
import { codemirror, codemirrorOption } from '../src/codemirror'
import exampleCode from './example'

export default {
  name: 'App',
  components: {
    codemirror
  },
  methods:{
    expandChange(val){
      console.log(val);
    }
  },
  data () {
    return {
      exampleCode: exampleCode,
      codemirrorOption: codemirrorOption,
      jsLabs: ['https://cdn.jsdelivr.net/npm/element-ui@2.12.0/lib/index.js'],
      cssLabs: [
        'https://cdn.jsdelivr.net/npm/element-ui@2.12.0/lib/theme-chalk/index.css'
      ],
      code: `<p>感谢使用~~<p>`
    }
  }
}
</script>

<style>
@import url('./app.css');
pre {
  line-height: 1.4;
  padding: 1.25rem 1.5rem;
  margin: 0.85rem 0;
  background-color: #282c34;
  border-radius: 6px;
  overflow: auto;
  color: rgba(255, 255, 255, 0.9);
}
</style>
