<template>
  <div class="containers">
    <div class="canvas" ref="canvas"></div>
    <div id="js-properties-panel" class="panel"></div>

    <div class="operate-box">
      <button class="save-btn" @click="saveDiagramToServer">保存给后端</button>
      <button class="save-btn" @click="saveDiagramToSvg">保存为svg文件</button>
      <a ref="saveDiagram"></a>
    </div>

  </div>
</template>

<script>
  import BpmnModeler from 'bpmn-js/lib/Modeler'
  import propertiesPanelModule from 'bpmn-js-properties-panel'
  import propertiesProviderModule from 'bpmn-js-properties-panel/lib/provider/camunda'
  import camundaModdleDescriptor from 'camunda-bpmn-moddle/resources/camunda'
  //汉化
  import customTranslate from "../components/customTranslate";
  //已有的流程图文件
  import {
    xmlStr
  } from '../mock/xmlStr' // 这里是直接引用了xml字符串

  export default {
    name: 'Home',
    components: {},
    mounted() {
      this.init()
    },
    data() {
      return {
        // bpmn建模器
        bpmnModeler: null,
        container: null,
        canvas: null
      }
    },
    methods: {
      init() {
        // 获取到属性ref为“canvas”的dom节点
        const canvas = this.$refs.canvas
        //汉化模块
        let customTranslateModule = {translate: ['value', customTranslate]}
        // 建模
        this.bpmnModeler = new BpmnModeler({
          container: canvas,
          //添加控制板
          propertiesPanel: {
            parent: '#js-properties-panel'
          },
          additionalModules: [
            // 右边的属性栏
            propertiesProviderModule,
            propertiesPanelModule,
            customTranslateModule
          ],
          moddleExtensions: {
            camunda: camundaModdleDescriptor
          }
        })

        this.createNewDiagram()
      },

      // 将xml字符串转换成图显示出来
      createNewDiagram() {
        this.bpmnModeler.importXML(xmlStr).then(res => {
          // 让图能自适应屏幕
          var canvas = this.bpmnModeler.get('canvas')
          canvas.zoom('fit-viewport')
        })
      },

      //保存为svg文件
      saveDiagramToSvg() {
        this.bpmnModeler.saveSVG({format: true}).then(res => {
          let str = res.svg
          // 把str转换为URI，下载要用到的
          const encodedData = encodeURIComponent(str)
          // 下载图的具体操作,改变a的属性，className令a标签可点击，href令能下载，download是下载的文件的名字
          const link = this.$refs.saveDiagram
          link.href = 'data:application/bpmn20-xml;charset=UTF-8,' + encodedData
          link.download = 'svg文件.svg'
          link.click()
        })
      },
      //保存为xml字符串传给后端
      saveDiagramToServer() {
        this.bpmnModeler.saveXML({format: true}).then(res => {
          console.log(res.xml)
        })
      }
    }
  }
</script>

<style scoped lang="scss">
  * {
    margin: 0;
    padding: 0;
  }

  input {
    box-sizing: border-box !important;
  }

  .containers {
    position: absolute;
    background-color: #ffffff;
    width: 100%;
    height: 100%;
  }

  .canvas {
    width: 100%;
    height: 100%;
  }

  .panel {
    position: absolute;
    right: 0;
    top: 0;
    width: 400px;
  }

  .operate-box {
    position: absolute;
    top: 0;
    left: 50%;
    .save-btn {

    }
  }

</style>

