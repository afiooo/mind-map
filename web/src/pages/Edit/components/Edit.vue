<template>
  <div
    class="editContainer"
    @dragenter.stop.prevent="onDragenter"
    @dragleave.stop.prevent
    @dragover.stop.prevent
    @drop.stop.prevent
  >
    <div
      class="mindMapContainer"
      id="mindMapContainer"
      ref="mindMapContainer"
    ></div>
    <Count :mindMap="mindMap" v-if="!isZenMode"></Count>
    <Navigator v-if="mindMap" :mindMap="mindMap"></Navigator>
    <NavigatorToolbar :mindMap="mindMap" v-if="!isZenMode"></NavigatorToolbar>
    <OutlineSidebar :mindMap="mindMap"></OutlineSidebar>
    <Style v-if="mindMap && !isZenMode" :mindMap="mindMap"></Style>
    <BaseStyle
      :data="mindMapData"
      :configData="mindMapConfig"
      :mindMap="mindMap"
    ></BaseStyle>
    <AssociativeLineStyle
      v-if="mindMap"
      :mindMap="mindMap"
    ></AssociativeLineStyle>
    <Theme v-if="mindMap" :data="mindMapData" :mindMap="mindMap"></Theme>
    <Structure :mindMap="mindMap"></Structure>
    <ShortcutKey></ShortcutKey>
    <Contextmenu v-if="mindMap" :mindMap="mindMap"></Contextmenu>
    <RichTextToolbar v-if="mindMap" :mindMap="mindMap"></RichTextToolbar>
    <NodeNoteContentShow
      v-if="mindMap"
      :mindMap="mindMap"
    ></NodeNoteContentShow>
    <NodeAttachment v-if="mindMap" :mindMap="mindMap"></NodeAttachment>
    <NodeImgPreview v-if="mindMap" :mindMap="mindMap"></NodeImgPreview>
    <SidebarTrigger v-if="!isZenMode"></SidebarTrigger>
    <Search v-if="mindMap" :mindMap="mindMap"></Search>
    <NodeIconSidebar v-if="mindMap" :mindMap="mindMap"></NodeIconSidebar>
    <NodeIconToolbar v-if="mindMap" :mindMap="mindMap"></NodeIconToolbar>
    <OutlineEdit v-if="mindMap" :mindMap="mindMap"></OutlineEdit>
    <Scrollbar v-if="isShowScrollbar && mindMap" :mindMap="mindMap"></Scrollbar>
    <FormulaSidebar v-if="mindMap" :mindMap="mindMap"></FormulaSidebar>
    <SourceCodeEdit v-if="mindMap" :mindMap="mindMap"></SourceCodeEdit>
    <NodeOuterFrame v-if="mindMap" :mindMap="mindMap"></NodeOuterFrame>
    <NodeTagStyle v-if="mindMap" :mindMap="mindMap"></NodeTagStyle>
    <Setting :configData="mindMapConfig" :mindMap="mindMap"></Setting>
    <NodeImgPlacementToolbar
      v-if="mindMap"
      :mindMap="mindMap"
    ></NodeImgPlacementToolbar>
    <NodeNoteSidebar v-if="mindMap" :mindMap="mindMap"></NodeNoteSidebar>
    <AiCreate v-if="mindMap && enableAi" :mindMap="mindMap"></AiCreate>
    <AiChat v-if="enableAi"></AiChat>
    <LinkNodeSelect
      v-if="mindMap && supportNodeLink"
      :mindMap="mindMap"
    ></LinkNodeSelect>
    <div
      class="dragMask"
      v-if="showDragMask"
      @dragleave.stop.prevent="onDragleave"
      @dragover.stop.prevent
      @drop.stop.prevent="onDrop"
    >
      <div class="dragTip">{{ $t('edit.dragTip') }}</div>
    </div>
  </div>
</template>

<script>
import MindMap from 'simple-mind-map'
import MiniMap from 'simple-mind-map/src/plugins/MiniMap.js'
import Watermark from 'simple-mind-map/src/plugins/Watermark.js'
import KeyboardNavigation from 'simple-mind-map/src/plugins/KeyboardNavigation.js'
import ExportPDF from 'simple-mind-map/src/plugins/ExportPDF.js'
import ExportXMind from 'simple-mind-map/src/plugins/ExportXMind.js'
import Export from 'simple-mind-map/src/plugins/Export.js'
import Drag from 'simple-mind-map/src/plugins/Drag.js'
import Select from 'simple-mind-map/src/plugins/Select.js'
import RichText from 'simple-mind-map/src/plugins/RichText.js'
import AssociativeLine from 'simple-mind-map/src/plugins/AssociativeLine.js'
import TouchEvent from 'simple-mind-map/src/plugins/TouchEvent.js'
import NodeImgAdjust from 'simple-mind-map/src/plugins/NodeImgAdjust.js'
import SearchPlugin from 'simple-mind-map/src/plugins/Search.js'
import Painter from 'simple-mind-map/src/plugins/Painter.js'
import ScrollbarPlugin from 'simple-mind-map/src/plugins/Scrollbar.js'
import Formula from 'simple-mind-map/src/plugins/Formula.js'
import RainbowLines from 'simple-mind-map/src/plugins/RainbowLines.js'
import Demonstrate from 'simple-mind-map/src/plugins/Demonstrate.js'
import OuterFrame from 'simple-mind-map/src/plugins/OuterFrame.js'
import MindMapLayoutPro from 'simple-mind-map/src/plugins/MindMapLayoutPro.js'
import NodeBase64ImageStorage from 'simple-mind-map/src/plugins/NodeBase64ImageStorage.js'
import Themes from 'simple-mind-map-plugin-themes'
// 协同编辑插件
// import Cooperate from 'simple-mind-map/src/plugins/Cooperate.js'
// 以下插件为付费插件，详情请查看开发文档。依次为：手绘风格插件、标记插件、编号插件、Freemind软件格式导入导出插件、Excel软件格式导入导出插件、待办插件、节点连线流动效果插件、动量效果插件、向右鱼骨图插件、节点链接插件、扩展节点形状插件、扩展主题列表插件
import HandDrawnLikeStyle from 'simple-mind-map-plugin-handdrawnlikestyle'
import Notation from 'simple-mind-map-plugin-notation'
import Numbers from 'simple-mind-map-plugin-numbers'
import Freemind from 'simple-mind-map-plugin-freemind'
import Excel from 'simple-mind-map-plugin-excel'
import Checkbox from 'simple-mind-map-plugin-checkbox'
import LineFlow from 'simple-mind-map-plugin-lineflow'
import Momentum from 'simple-mind-map-plugin-momentum'
import RightFishbone from 'simple-mind-map-plugin-right-fishbone'
import NodeLink from 'simple-mind-map-plugin-node-link'
// import MoreShapes from 'simple-mind-map-plugin-more-shapes'
// import MoreThemes from 'simple-mind-map-plugin-more-themes'
// npm link simple-mind-map simple-mind-map-plugin-excel simple-mind-map-plugin-freemind simple-mind-map-plugin-numbers simple-mind-map-plugin-notation simple-mind-map-plugin-handdrawnlikestyle simple-mind-map-plugin-checkbox simple-mind-map-plugin-lineflow simple-mind-map-plugin-momentum simple-mind-map-plugin-right-fishbone simple-mind-map-plugin-node-link
// simple-mind-map-plugin-themes
// simple-mind-map-plugin-more-themes simple-mind-map-plugin-more-shapes
import OutlineSidebar from './OutlineSidebar.vue'
import Style from './Style.vue'
import BaseStyle from './BaseStyle.vue'
import Theme from './Theme.vue'
import Structure from './Structure.vue'
import Count from './Count.vue'
import NavigatorToolbar from './NavigatorToolbar.vue'
import ShortcutKey from './ShortcutKey.vue'
import Contextmenu from './Contextmenu.vue'
import RichTextToolbar from './RichTextToolbar.vue'
import NodeNoteContentShow from './NodeNoteContentShow.vue'
import { getData, getConfig, storeData } from '@/api'
import Navigator from './Navigator.vue'
import NodeImgPreview from './NodeImgPreview.vue'
import SidebarTrigger from './SidebarTrigger.vue'
import { mapState } from 'vuex'
import icon from '@/config/icon'
import CustomNodeContent from './CustomNodeContent.vue'
import Color from './Color.vue'
import Vue from 'vue'
import router from '../../../router'
import store from '../../../store'
import i18n from '../../../i18n'
import Search from './Search.vue'
import NodeIconSidebar from './NodeIconSidebar.vue'
import NodeIconToolbar from './NodeIconToolbar.vue'
import OutlineEdit from './OutlineEdit.vue'
import { showLoading, hideLoading } from '@/utils/loading'
import handleClipboardText from '@/utils/handleClipboardText'
import { getParentWithClass } from '@/utils'
import Scrollbar from './Scrollbar.vue'
import exampleData from 'simple-mind-map/example/exampleData'
import FormulaSidebar from './FormulaSidebar.vue'
import SourceCodeEdit from './SourceCodeEdit.vue'
import NodeAttachment from './NodeAttachment.vue'
import NodeOuterFrame from './NodeOuterFrame.vue'
import NodeTagStyle from './NodeTagStyle.vue'
import Setting from './Setting.vue'
import AssociativeLineStyle from './AssociativeLineStyle.vue'
import NodeImgPlacementToolbar from './NodeImgPlacementToolbar.vue'
import NodeNoteSidebar from './NodeNoteSidebar.vue'
import AiCreate from './AiCreate.vue'
import AiChat from './AiChat.vue'
import LinkNodeSelect from './LinkNodeSelect.vue'

// 注册插件
MindMap.usePlugin(MiniMap)
  .usePlugin(Watermark)
  .usePlugin(Drag)
  .usePlugin(KeyboardNavigation)
  .usePlugin(ExportPDF)
  .usePlugin(ExportXMind)
  .usePlugin(Export)
  .usePlugin(Select)
  .usePlugin(AssociativeLine)
  .usePlugin(NodeImgAdjust)
  .usePlugin(TouchEvent)
  .usePlugin(SearchPlugin)
  .usePlugin(Painter)
  .usePlugin(Formula)
  .usePlugin(RainbowLines)
  .usePlugin(Demonstrate)
  .usePlugin(OuterFrame)
  .usePlugin(MindMapLayoutPro)
  .usePlugin(NodeBase64ImageStorage)
// .usePlugin(Cooperate) // 协同插件

// 注册主题
Themes.init(MindMap)
// 扩展主题列表
if (typeof MoreThemes !== 'undefined') {
  MoreThemes.init(MindMap)
}

export default {
  components: {
    OutlineSidebar,
    Style,
    BaseStyle,
    Theme,
    Structure,
    Count,
    NavigatorToolbar,
    ShortcutKey,
    Contextmenu,
    RichTextToolbar,
    NodeNoteContentShow,
    Navigator,
    NodeImgPreview,
    SidebarTrigger,
    Search,
    NodeIconSidebar,
    NodeIconToolbar,
    OutlineEdit,
    Scrollbar,
    FormulaSidebar,
    SourceCodeEdit,
    NodeAttachment,
    NodeOuterFrame,
    NodeTagStyle,
    Setting,
    AssociativeLineStyle,
    NodeImgPlacementToolbar,
    NodeNoteSidebar,
    AiCreate,
    AiChat,
    LinkNodeSelect
  },
  data() {
    return {
      enableShowLoading: true,
      mindMap: null,
      mindMapData: null,
      mindMapConfig: {},
      prevImg: '',
      storeConfigTimer: null,
      showDragMask: false
    }
  },
  computed: {
    ...mapState({
      isZenMode: state => state.localConfig.isZenMode,
      openNodeRichText: state => state.localConfig.openNodeRichText,
      isShowScrollbar: state => state.localConfig.isShowScrollbar,
      enableDragImport: state => state.localConfig.enableDragImport,
      useLeftKeySelectionRightKeyDrag: state =>
        state.localConfig.useLeftKeySelectionRightKeyDrag,
      isUseHandDrawnLikeStyle: state =>
        state.localConfig.isUseHandDrawnLikeStyle,
      isUseMomentum: state => state.localConfig.isUseMomentum,
      extraTextOnExport: state => state.extraTextOnExport,
      isDragOutlineTreeNode: state => state.isDragOutlineTreeNode,
      enableAi: state => state.localConfig.enableAi,
      supportNodeLink: state => state.supportNodeLink
    })
  },
  watch: {
    openNodeRichText() {
      if (this.openNodeRichText) {
        this.addRichTextPlugin()
      } else {
        this.removeRichTextPlugin()
      }
    },
    isShowScrollbar() {
      if (this.isShowScrollbar) {
        this.addScrollbarPlugin()
      } else {
        this.removeScrollbarPlugin()
      }
    },
    isUseHandDrawnLikeStyle() {
      if (this.isUseHandDrawnLikeStyle) {
        this.addHandDrawnLikeStylePlugin()
      } else {
        this.removeHandDrawnLikeStylePlugin()
      }
    },
    isUseMomentum() {
      if (this.isUseMomentum) {
        this.addMomentumPlugin()
      } else {
        this.removeMomentumPlugin()
      }
    }
  },
  mounted() {
    showLoading()
    // this.showNewFeatureInfo()
    this.getData()
    this.init()
    this.$bus.$on('execCommand', this.execCommand)
    this.$bus.$on('paddingChange', this.onPaddingChange)
    this.$bus.$on('export', this.export)
    this.$bus.$on('setData', this.setData)
    this.$bus.$on('startTextEdit', this.handleStartTextEdit)
    this.$bus.$on('endTextEdit', this.handleEndTextEdit)
    this.$bus.$on('createAssociativeLine', this.handleCreateLineFromActiveNode)
    this.$bus.$on('startPainter', this.handleStartPainter)
    this.$bus.$on('node_tree_render_end', this.handleHideLoading)
    this.$bus.$on('showLoading', this.handleShowLoading)
    this.$bus.$on('localStorageExceeded', this.onLocalStorageExceeded)
    window.addEventListener('resize', this.handleResize)
    document.body.addEventListener('click', this.onVipCheckClick)
    this.$bus.$on('showDownloadTip', this.showDownloadTip)
    this.$bus.$on('vipCheckClick', this.onVipCheckClick)
  },
  beforeDestroy() {
    this.$bus.$off('execCommand', this.execCommand)
    this.$bus.$off('paddingChange', this.onPaddingChange)
    this.$bus.$off('export', this.export)
    this.$bus.$off('setData', this.setData)
    this.$bus.$off('startTextEdit', this.handleStartTextEdit)
    this.$bus.$off('endTextEdit', this.handleEndTextEdit)
    this.$bus.$off('createAssociativeLine', this.handleCreateLineFromActiveNode)
    this.$bus.$off('startPainter', this.handleStartPainter)
    this.$bus.$off('node_tree_render_end', this.handleHideLoading)
    this.$bus.$off('showLoading', this.handleShowLoading)
    this.$bus.$off('localStorageExceeded', this.onLocalStorageExceeded)
    window.removeEventListener('resize', this.handleResize)
    document.body.removeEventListener('click', this.onVipCheckClick)
    this.$bus.$off('showDownloadTip', this.showDownloadTip)
    this.$bus.$off('vipCheckClick', this.onVipCheckClick)
    this.mindMap.destroy()
  },
  methods: {
    onLocalStorageExceeded() {
      this.$notify({
        type: 'warning',
        title: this.$t('edit.tip'),
        message: this.$t('edit.localStorageExceededTip'),
        duration: 0
      })
    },

    handleStartTextEdit() {
      this.mindMap.renderer.startTextEdit()
    },

    handleEndTextEdit() {
      this.mindMap.renderer.endTextEdit()
    },

    handleCreateLineFromActiveNode() {
      this.mindMap.associativeLine.createLineFromActiveNode()
    },

    handleStartPainter() {
      this.mindMap.painter.startPainter()
    },

    handleResize() {
      this.mindMap.resize()
    },

    // 显示loading
    handleShowLoading() {
      this.enableShowLoading = true
      showLoading()
    },

    // 渲染结束后关闭loading
    handleHideLoading() {
      if (this.enableShowLoading) {
        this.enableShowLoading = false
        hideLoading()
      }
    },

    // 获取思维导图数据，实际应该调接口获取
    getData() {
      this.mindMapData = getData()
      this.mindMapConfig = getConfig() || {}
    },

    // 存储数据当数据有变时
    bindSaveEvent() {
      this.$bus.$on('data_change', data => {
        storeData({ root: data })
      })
      this.$bus.$on('view_data_change', data => {
        clearTimeout(this.storeConfigTimer)
        this.storeConfigTimer = setTimeout(() => {
          storeData({
            view: data
          })
        }, 300)
      })
    },

    // 手动保存
    manualSave() {
      storeData(this.mindMap.getData(true))
    },

    // 初始化
    init() {
      let hasFileURL = this.hasFileURL()
      let { root, layout, theme, view } = this.mindMapData
      const config = this.mindMapConfig
      // 如果url中存在要打开的文件，那么思维导图数据、主题、布局都使用默认的
      if (hasFileURL) {
        root = {
          data: {
            text: this.$t('edit.root')
          },
          children: []
        }
        layout = exampleData.layout
        theme = exampleData.theme
        view = null
      }
      this.mindMap = new MindMap({
        el: this.$refs.mindMapContainer,
        data: root,
        fit: false,
        layout: layout,
        theme: theme.template,
        themeConfig: theme.config,
        viewData: view,
        nodeTextEditZIndex: 1000,
        nodeNoteTooltipZIndex: 1000,
        customNoteContentShow: {
          show: (content, left, top, node) => {
            this.$bus.$emit('showNoteContent', content, left, top, node)
          },
          hide: () => {
            // this.$bus.$emit('hideNoteContent')
          }
        },
        openRealtimeRenderOnNodeTextEdit: true,
        enableAutoEnterTextEditWhenKeydown: true,
        demonstrateConfig: {
          openBlankMode: true
        },
        ...(config || {}),
        iconList: [...icon],
        useLeftKeySelectionRightKeyDrag: this.useLeftKeySelectionRightKeyDrag,
        customInnerElsAppendTo: null,
        customHandleClipboardText: handleClipboardText,
        defaultNodeImage: require('../../../assets/img/图片加载失败.svg'),
        initRootNodePosition: ['center', 'center'],
        handleIsSplitByWrapOnPasteCreateNewNode: () => {
          return this.$confirm(
            this.$t('edit.splitByWrap'),
            this.$t('edit.tip'),
            {
              confirmButtonText: this.$t('edit.yes'),
              cancelButtonText: this.$t('edit.no'),
              type: 'warning'
            }
          )
        },
        errorHandler: (code, err) => {
          console.error(err)
          switch (code) {
            case 'export_error':
              this.$message.error(this.$t('edit.exportError'))
              break
            default:
              break
          }
        },
        addContentToFooter: () => {
          const text = this.extraTextOnExport.trim()
          if (!text) return null
          const el = document.createElement('div')
          el.className = 'footer'
          el.innerHTML = text
          const cssText = `
            .footer {
              width: 100%;
              height: 30px;
              display: flex;
              justify-content: center;
              align-items: center;
              font-size: 12px;
              color: #979797;
            }
          `
          return {
            el,
            cssText,
            height: 30
          }
        },
        expandBtnNumHandler: num => {
          return num >= 100 ? '…' : num
        },
        beforeDeleteNodeImg: node => {
          return new Promise(resolve => {
            this.$confirm(
              this.$t('edit.deleteNodeImgTip'),
              this.$t('edit.tip'),
              {
                confirmButtonText: this.$t('edit.yes'),
                cancelButtonText: this.$t('edit.no'),
                type: 'warning'
              }
            )
              .then(() => {
                resolve(false)
              })
              .catch(() => {
                resolve(true)
              })
          })
        }
        // createNodePrefixContent: node => {
        //   const el = document.createElement('div')
        //   el.style.width = '50px'
        //   el.style.height = '50px'
        //   el.style.background = 'red'
        //   return {
        //     el,
        //     width: 50,
        //     height: 50
        //   }
        // },
        // createNodePostfixContent: node => {
        //   const domparser = new DOMParser()
        //   const doc = domparser.parseFromString(
        //     '<b style="background-color: rgb(214, 239, 214);">白日依山尽</b>',
        //     'text/html'
        //   )
        //   const el = doc.querySelector('b')
        //   return {
        //     el,
        //     width: 50,
        //     height: 50
        //   }
        // },
        // addContentToHeader: () => {
        //   const el = document.createElement('div')
        //   el.className = 'footer'
        //   el.innerHTML = '理想青年实验室'
        //   const cssText = `
        //     .header {
        //       width: 100%;
        //       height: 50px;
        //       background: #f5f5f5;
        //       display: flex;
        //       justify-content: center;
        //       align-items: center
        //     }
        //   `
        //   return {
        //     el,
        //     cssText,
        //     height: 50
        //   }
        // },
        // beforeShortcutRun: (key, activeNodeList) => {
        //   console.log(key, activeNodeList)
        //   // 阻止删除快捷键行为
        //   if (key === 'Backspace') {
        //     return true
        //   }
        // }
        // handleNodePasteImg: img => {
        //   console.log(img)
        //   return new Promise(resolve => {
        //     setTimeout(() => {
        //       resolve({
        //         url: require('../../../assets/img/themes/autumn.jpg'),
        //         size: {
        //           width: 100,
        //           height: 100
        //         }
        //       })
        //     }, 200)
        //   })
        // }
        // isUseCustomNodeContent: true,
        // 示例1：组件里用到了router、store、i18n等实例化vue组件时需要用到的东西
        // customCreateNodeContent: (node) => {
        //   let el = document.createElement('div')
        //   let Comp = Vue.extend(Color)
        //   let comp = new Comp({
        //     router,
        //     store,
        //     i18n
        //   })
        //   comp.$mount(el)
        //   return comp.$el
        // },
        // 示例2：组件里没有用到示例1的东西
        // customCreateNodeContent: (node) => {
        //   let el = document.createElement('div')
        //   let Comp = Vue.extend(CustomNodeContent)
        //   let comp = new Comp({
        //     propsData: {
        //       html: node.nodeData.data.text
        //     }
        //   })
        //   comp.$mount(el)
        //   return comp.$el
        // },
        // 示例3：普通元素
        // customCreateNodeContent: node => {
        //   let el = document.createElement('div')
        //   el.style.cssText = `
        //     width: 203px;
        //     height: 78px;
        //     opacity: 0.8;
        //     background-image: linear-gradient(0deg, rgba(53,130,172,0.06) 0%, rgba(24,75,116,0.06) 100%);
        //     box-shadow: inset 0 1px 15px 0 rgba(119,196,255,0.40);
        //     border-radius: 2px;
        //     display: flex;
        //     justify-content: center;
        //     align-items: center;
        //   `
        //   el.innerHTML = `
        //     ${node.nodeData.data.text}
        //     <img crossOrigin="anonymous" src="/img/cactus.jpg" />
        //   `
        //   return el
        // }
      })
      this.loadPlugins()
      this.mindMap.keyCommand.addShortcut('Control+s', () => {
        this.manualSave()
      })
      // 转发事件
      ;[
        'node_active',
        'data_change',
        'view_data_change',
        'back_forward',
        'node_contextmenu',
        'node_click',
        'draw_click',
        'expand_btn_click',
        'svg_mousedown',
        'mouseup',
        'mode_change',
        'node_tree_render_end',
        'rich_text_selection_change',
        'transforming-dom-to-images',
        'generalization_node_contextmenu',
        'painter_start',
        'painter_end',
        'scrollbar_change',
        'scale',
        'translate',
        'node_attachmentClick',
        'node_attachmentContextmenu',
        'demonstrate_jump',
        'exit_demonstrate',
        'node_note_dblclick',
        'node_mousedown'
      ].forEach(event => {
        this.mindMap.on(event, (...args) => {
          this.$bus.$emit(event, ...args)
        })
      })
      this.bindSaveEvent()
      this.testDynamicCreateNodes()
      // 如果应用被接管，那么抛出事件传递思维导图实例
      if (window.takeOverApp) {
        this.$bus.$emit('app_inited', this.mindMap)
      }
      // 解析url中的文件
      if (hasFileURL) {
        this.$bus.$emit('handle_file_url')
      }
      // api/index.js文件使用
      // 当正在编辑本地文件时通过该方法获取最新数据
      Vue.prototype.getCurrentData = () => {
        const fullData = this.mindMap.getData(true)
        return { ...fullData }
      }
      // 协同测试
      this.cooperateTest()
      // 销毁
      // setTimeout(() => {
      //   console.log('销毁')
      //   this.mindMap.destroy()
      // }, 10000)
      // 测试
      // setTimeout(() => {
      //   console.log(this.mindMap.renderer.root.getRect())
      //   console.log(this.mindMap.renderer.root.getRectInSvg())
      // }, 5000);
      // setTimeout(() => {
      //   this.mindMap.renderer.renderTree.data.fillColor = 'red'
      //   this.mindMap.render()
      //   this.mindMap.reRender()
      //   this.mindMap.render()
      // }, 5000)
    },

    // 加载相关插件
    loadPlugins() {
      if (this.openNodeRichText) this.addRichTextPlugin()
      if (this.isShowScrollbar) this.addScrollbarPlugin()
      if (typeof HandDrawnLikeStyle !== 'undefined') {
        this.$store.commit('setSupportHandDrawnLikeStyle', true)
        if (this.isUseHandDrawnLikeStyle) this.addHandDrawnLikeStylePlugin()
      }
      if (typeof Momentum !== 'undefined') {
        this.$store.commit('setSupportMomentum', true)
        if (this.isUseMomentum) this.addMomentumPlugin()
      }
      if (typeof Notation !== 'undefined') {
        this.mindMap.addPlugin(Notation)
        this.$store.commit('setSupportMark', true)
      }
      if (typeof Numbers !== 'undefined') {
        this.mindMap.addPlugin(Numbers)
        this.$store.commit('setSupportNumbers', true)
      }
      if (typeof Freemind !== 'undefined') {
        this.mindMap.addPlugin(Freemind)
        this.$store.commit('setSupportFreemind', true)
        Vue.prototype.Freemind = Freemind
      }
      if (typeof Excel !== 'undefined') {
        this.mindMap.addPlugin(Excel)
        this.$store.commit('setSupportExcel', true)
        Vue.prototype.Excel = Excel
      }
      if (typeof Checkbox !== 'undefined') {
        this.mindMap.addPlugin(Checkbox)
        this.$store.commit('setSupportCheckbox', true)
      }
      if (typeof LineFlow !== 'undefined') {
        this.mindMap.addPlugin(LineFlow)
        this.$store.commit('setSupportLineFlow', true)
      }
      if (typeof RightFishbone !== 'undefined') {
        this.mindMap.addPlugin(RightFishbone)
        this.$store.commit('setSupportRightFishbone', true)
      }
      if (typeof NodeLink !== 'undefined') {
        this.mindMap.addPlugin(NodeLink)
        this.$store.commit('setSupportNodeLink', true)
      }
      if (typeof MoreShapes !== 'undefined') {
        this.mindMap.addPlugin(MoreShapes)
        this.$store.commit('setSupportMoreShapes', true)
      }
      // 扩展侧边主题列表
      if (typeof MoreThemes !== 'undefined') {
        const extendThemeGroupList = [
          {
            name: this.$t('edit.withBg'), // 主题组名称
            // 主题列表
            list: [...MoreThemes.lightList, ...MoreThemes.darkList].map(
              item => {
                return {
                  ...item,
                  img: MoreThemes.themeImgMap[item.value]
                }
              }
            )
          }
        ]
        this.$store.commit('setExtendThemeGroupList', extendThemeGroupList)
        this.$store.commit('setBgList', MoreThemes.bgList)
      }
    },

    // url中是否存在要打开的文件
    hasFileURL() {
      const fileURL = this.$route.query.fileURL
      if (!fileURL) return false
      return /\.(smm|json|xmind|md|xlsx)$/.test(fileURL)
    },

    // 动态设置思维导图数据
    setData(data) {
      this.handleShowLoading()
      let rootNodeData = null
      if (data.root) {
        this.mindMap.setFullData(data)
        rootNodeData = data.root
      } else {
        this.mindMap.setData(data)
        rootNodeData = data
      }
      this.mindMap.view.reset()
      this.manualSave()
      // 如果导入的是富文本内容，那么自动开启富文本模式
      if (rootNodeData.data.richText && !this.openNodeRichText) {
        this.$bus.$emit('toggleOpenNodeRichText', true)
        this.$notify.info({
          title: this.$t('edit.tip'),
          message: this.$t('edit.autoOpenNodeRichTextTip')
        })
      }
    },

    // 重新渲染
    reRender() {
      this.mindMap.reRender()
    },

    // 执行命令
    execCommand(...args) {
      this.mindMap.execCommand(...args)
    },

    // 导出
    async export(...args) {
      try {
        showLoading()
        await this.mindMap.export(...args)
        hideLoading()
      } catch (error) {
        console.log(error)
        hideLoading()
      }
    },

    // 修改导出内边距
    onPaddingChange(data) {
      this.mindMap.updateConfig(data)
    },

    // 显示新特性提示
    showNewFeatureInfo() {
      let showed = localStorage.getItem('SIMPLE_MIND_MAP_NEW_FEATURE_TIP_1')
      if (!showed) {
        this.$notify.info({
          title: this.$t('edit.newFeatureNoticeTitle'),
          message: this.$t('edit.newFeatureNoticeMessage'),
          duration: 0,
          onClose: () => {
            localStorage.setItem('SIMPLE_MIND_MAP_NEW_FEATURE_TIP_1', true)
          }
        })
      }
    },

    // 加载节点富文本编辑插件
    addRichTextPlugin() {
      if (!this.mindMap) return
      this.mindMap.addPlugin(RichText)
    },

    // 移除节点富文本编辑插件
    removeRichTextPlugin() {
      this.mindMap.removePlugin(RichText)
    },

    // 加载滚动条插件
    addScrollbarPlugin() {
      if (!this.mindMap) return
      this.mindMap.addPlugin(ScrollbarPlugin)
    },

    // 移除滚动条插件
    removeScrollbarPlugin() {
      this.mindMap.removePlugin(ScrollbarPlugin)
    },

    // 加载手绘风格插件
    addHandDrawnLikeStylePlugin() {
      try {
        if (!this.mindMap) return
        this.mindMap.addPlugin(HandDrawnLikeStyle)
        this.mindMap.reRender()
      } catch (error) {
        console.log('手绘风格插件不存在')
      }
    },

    // 移除手绘风格插件
    removeHandDrawnLikeStylePlugin() {
      try {
        this.mindMap.removePlugin(HandDrawnLikeStyle)
        this.mindMap.reRender()
      } catch (error) {
        console.log('手绘风格插件不存在')
      }
    },

    // 加载动量效果插件
    addMomentumPlugin() {
      try {
        if (!this.mindMap) return
        this.mindMap.addPlugin(Momentum)
      } catch (error) {
        console.log('动量效果插件不存在')
      }
    },

    // 移除动量效果插件
    removeMomentumPlugin() {
      try {
        this.mindMap.removePlugin(Momentum)
      } catch (error) {
        console.log('动量效果插件不存在')
      }
    },

    // 测试动态插入节点
    testDynamicCreateNodes() {
      // return
      setTimeout(() => {
        // 动态给指定节点添加子节点
        // this.mindMap.execCommand(
        //   'INSERT_CHILD_NODE',
        //   false,
        //   null,
        //   {
        //     text: '自定义内容'
        //   },
        //   [
        //     {
        //       data: {
        //         text: '自定义子节点'
        //       }
        //     }
        //   ]
        // )
        // 动态给指定节点添加同级节点
        // this.mindMap.execCommand(
        //   'INSERT_NODE',
        //   false,
        //   null,
        //   {
        //     text: '自定义内容'
        //   },
        //   [
        //     {
        //       data: {
        //         text: '自定义同级节点'
        //       },
        //       children: [
        //         {
        //           data: {
        //             text: '自定义同级节点2'
        //           },
        //           children: []
        //         }
        //       ]
        //     }
        //   ]
        // )
        // 动态插入多个子节点
        // this.mindMap.execCommand('INSERT_MULTI_CHILD_NODE', null, [
        //   {
        //     data: {
        //       text: '自定义节点1'
        //     },
        //     children: [
        //       {
        //         data: {
        //           text: '自定义节点1-1'
        //         },
        //         children: []
        //       }
        //     ]
        //   },
        //   {
        //     data: {
        //       text: '自定义节点2'
        //     },
        //     children: [
        //       {
        //         data: {
        //           text: '自定义节点2-1'
        //         },
        //         children: []
        //       }
        //     ]
        //   }
        // ])
        // 动态插入多个同级节点
        // this.mindMap.execCommand('INSERT_MULTI_NODE', null, [
        //   {
        //     data: {
        //       text: '自定义节点1'
        //     },
        //     children: [
        //       {
        //         data: {
        //           text: '自定义节点1-1'
        //         },
        //         children: []
        //       }
        //     ]
        //   },
        //   {
        //     data: {
        //       text: '自定义节点2'
        //     },
        //     children: [
        //       {
        //         data: {
        //           text: '自定义节点2-1'
        //         },
        //         children: []
        //       }
        //     ]
        //   }
        // ])
        // 动态删除指定节点
        // this.mindMap.execCommand('REMOVE_NODE', this.mindMap.renderer.root.children[0])
      }, 5000)
    },

    // 协同测试
    cooperateTest() {
      if (this.mindMap.cooperate && this.$route.query.userName) {
        this.mindMap.cooperate.setProvider(null, {
          roomName: 'demo-room',
          signalingList: ['ws://localhost:4444']
        })
        this.mindMap.cooperate.setUserInfo({
          id: Math.random(),
          name: this.$route.query.userName,
          color: ['#409EFF', '#67C23A', '#E6A23C', '#F56C6C', '#909399'][
            Math.floor(Math.random() * 5)
          ],
          avatar:
            Math.random() > 0.5
              ? 'https://img0.baidu.com/it/u=4270674549,2416627993&fm=253&app=138&size=w931&n=0&f=JPEG&fmt=auto?sec=1696006800&t=4d32871d14a7224a4591d0c3c7a97311'
              : ''
        })
      }
    },

    // 拖拽文件到页面导入
    onDragenter() {
      if (!this.enableDragImport || this.isDragOutlineTreeNode) return
      this.showDragMask = true
    },
    onDragleave() {
      this.showDragMask = false
    },
    onDrop(e) {
      if (!this.enableDragImport) return
      this.showDragMask = false
      const dt = e.dataTransfer
      const file = dt.files && dt.files[0]
      if (!file) return
      this.$bus.$emit('importFile', file)
    },

    // 网页版功能试用提示
    onVipCheckClick(e) {
      const el = getParentWithClass(e.target, 'vip')
      if (el) {
        const className = el.classList.value.split(/\s+/).join('_')
        const storageKey = 'VIP_USAGE_TIP'
        let data = localStorage.getItem(storageKey)
        if (data) {
          data = JSON.parse(data)
        } else {
          data = {}
        }
        if (!data[className]) {
          data[className] = 0
        }
        data[className]++
        if (data[className] > 3) {
          this.showDownloadTip(
            this.$t('edit.tryTipTitle'),
            this.$t('edit.tryTipDesc')
          )
        }
        localStorage.setItem(storageKey, JSON.stringify(data))
      }
    },

    showDownloadTip(title, desc) {
      const h = this.$createElement
      this.$msgbox({
        title,
        message: h('div', null, [
          h('p', null, desc),
          h('div', null, [
            h(
              'a',
              {
                attrs: {
                  href:
                    'https://pan.baidu.com/s/1huasEbKsGNH2Af68dvWiOg?pwd=3bp3',
                  target: '_blank'
                },
                style: {
                  color: '#409eff',
                  marginRight: '12px'
                }
              },
              this.$t('edit.downBaidu')
            ),
            h(
              'a',
              {
                attrs: {
                  href: 'https://github.com/wanglin2/mind-map/releases',
                  target: '_blank'
                },
                style: {
                  color: '#409eff'
                }
              },
              this.$t('edit.downGithub')
            )
          ])
        ]),
        showCancelButton: false,
        showConfirmButton: false
      })
    }
  }
}
</script>

<style lang="less" scoped>
.editContainer {
  position: fixed;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;

  .dragMask {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(255, 255, 255, 0.8);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 3999;

    .dragTip {
      pointer-events: none;
      font-weight: bold;
    }
  }

  .mindMapContainer {
    position: absolute;
    left: 0px;
    top: 0px;
    width: 100%;
    height: 100%;

    // left: 100px;
    // top: 100px;
    // right: 100px;
    // bottom: 100px;
  }
}
</style>
