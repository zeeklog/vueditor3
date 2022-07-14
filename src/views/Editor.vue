<template>
  <div :id="id" class="editor"></div>
</template>

<script>
/**
 * 事件订阅发布,取代原始Event对象,提升IE下的兼容性
 * @overwite LoadEvent
 * @function on 创建监听事件
 * @function emit 触发监听事件
 * **/
class LoadEvent {
  constructor() {
    this.listeners = {}
  }
  on(eventName, callback) {
    if (this.listeners[eventName] === undefined) {
      this.listeners[eventName] = []
    }
    this.listeners[eventName].push(callback)
  }
  emit(eventName) {
    this.listeners[eventName] &&
      this.listeners[eventName].forEach(callback => callback())
  }
}

export default {
  name: 'vueEditor3',
  data() {
    return {
      // 单个组件可能存在多个不可复用的实例
      id: 'HC' + Math.round(Math.random() * 100),
      editor: null,
      defaultConfig: {
        UEDITOR_HOME_URL: '../ueditor/',
        enableAutoSave: false
      }
    }
  },
  props: {
    modelValue: {
      type: String,
      default: '暂无数据'
    },
    config: {
      type: Object,
      default: function () {
        return {}
      }
    },
    configPath: {
      type: String,
      default: '/ueditor/config/ueditor.config.js'
    },
    init: {
      type: Function,
      default: function () {
        return () => {}
      }
    },
    destroy: Boolean
  },
  computed: {
    // 混入配置文件
    mixedConfig() {
      return Object.assign({}, this.defaultConfig, this.config)
    }
  },
  methods: {
    /**
     * 按钮注册
     * @description 通过这个方法可以自定义注册功能按钮
     *  @param name 按钮注册名
     *  @param icon 按钮图标
     *  @param tip 按钮tips提示
     *  @param handler 按钮执行handler
     *  @param UE 全局UE对象，默认为window.UE
     * **/
    registerButton: ({ name, icon, tip, handler, UE = window.UE }) => {
      UE.registerUI(name, (editor, name) => {
        // 配置按钮需要执行的命令
        editor.registerCommand(name, {
          execCommand: () => {
            handler(editor, name)
          }
        })
        const btn = new UE.ui.Button({
          name,
          title: tip,
          // 自定义按钮的样式
          // 注意：ueditor原图标使用的是雪碧图， 自定义按钮需要自定义图标，设置background-image
          cssRules: `background-image: url(${icon}) !important;background-size: cover;`,
          onclick() {
            editor.execCommand(name)
          }
        })
        // 当选择事件selectionchange触发时，设置按钮的禁用和已选择状态
        editor.addListener('selectionchange', () => {
          const state = editor.queryCommandState(name)
          if (state === -1) {
            btn.setDisabled(true) // 禁用
            btn.setChecked(false) // 已选
          } else {
            btn.setDisabled(false)
            btn.setChecked(state)
          }
        })
        return btn
      })
    },
    // 实例化编辑器之前-JS依赖检测
    _beforeInitEditor(value) {
      // 准确判断ueditor.config.js和ueditor.all.js均已加载
      // 仅加载完ueditor.config.js时UE对象和UEDITOR_CONFIG对象也存在,
      // 仅加载完ueditor.all.js时UEDITOR_CONFIG对象也存在,但为空对象
      !!window.UE &&
      !!window.UEDITOR_CONFIG &&
      Object.keys(window.UEDITOR_CONFIG).length !== 0 &&
      !!window.UE.getEditor
        ? this._initEditor(value)
        : this._loadScripts().then(() => this._initEditor(value))
    },
    // 实例化编辑器
    _initEditor(value) {
      this.$nextTick(() => {
        this.init()
        // 没有按官网示例那样链式调用ready方法的原因在于需要拿到getEditor返回的实例
        this.editor = window.UE.getEditor(this.id, this.mixedConfig)
        this.editor.addListener('ready', () => {
          this.$emit('ready', this.editor)
          this.editor.setContent(value)
          // 设置请求头， 拉服务器的配置文件（上传、模板等接口配置）
          this.editor.execCommand('serverparam', function () {
            return {
              // 设置身份令牌
              // TODO 这里要和后端按沟通， 是使用当前登陆身份的令牌还是单独给一个
              auth_token: localStorage.getItem('token')
            }
          })
          this.editor.queryCommandValue('serverparam')
          this.editor.addListener('contentChange', () => {
            // 向父元素广播变更事件，子组件 -> 父组件 传值
            setTimeout(() => {
              this.$emit('update:modelValue', this.editor.getContent())
              this.$emit('puretxt', this.editor.getContentTxt())
            }, 100)
          })
        })
      })
    },
    // 动态添加JS依赖
    _loadScripts() {
      // 确保多个实例同时渲染时不会重复创建SCRIPT标签
      if (window.loadEnv) {
        return new Promise(resolve => {
          window.loadEnv.on('scriptsLoaded', function () {
            resolve()
          })
        })
      } else {
        // 不存在实例， 需要重新走一遍加载配置、加载js主包的流程
        window.loadEnv = new LoadEvent()
        return new Promise(resolve => {
          // 如果在其他地方只引用ueditor.all.min.js，
          // 在加载ueditor.config.js之后仍需要重新加载ueditor.all.min.js，所以必须确保ueditor.config.js已加载
          this._loadConfig()
            .then(() => this._loadCore())
            .then(() => {
              window.loadEnv.emit('scriptsLoaded')
              resolve()
            })
        })
      }
    },
    // 加载Ueditor的配置文件和生成windows实例
    _loadConfig() {
      return new Promise(resolve => {
        if (
          !!window.UE &&
          !!window.UEDITOR_CONFIG &&
          Object.keys(window.UEDITOR_CONFIG).length !== 0
        ) {
          resolve()
          return
        }
        const configScript = document.createElement('script')
        configScript.type = 'text/javascript'
        configScript.src = this.configPath
        document.getElementsByTagName('head')[0].appendChild(configScript)
        configScript.onload = function () {
          if (
            !!window.UE &&
            !!window.UEDITOR_CONFIG &&
            Object.keys(window.UEDITOR_CONFIG).length !== 0
          ) {
            resolve()
          } else {
            console.error('UE配置文件加载失败，请检查配置文件路径是否正确。')
          }
        }
      })
    },
    // 加载核心ueditor.all.js代码
    _loadCore() {
      return new Promise(resolve => {
        if (!!window.UE && !!window.UE.getEditor) {
          return resolve()
        }
        const coreScript = document.createElement('script')
        coreScript.type = 'text/javascript'
        coreScript.src = this.mixedConfig.UEDITOR_HOME_URL + 'ueditor.all.js'
        document.getElementsByTagName('head')[0].appendChild(coreScript)
        coreScript.onload = function () {
          if (!!window.UE && !!window.UE.getEditor) {
            resolve()
          } else {
            // console && //console.error('加载ueditor.all.min.js失败,请检查您的配置地址UEDITOR_HOME_URL填写是否正确!')
          }
        }
      })
    },
    // 设置编辑器内的内容
    _setContent(value) {
      value === this.editor.getContent() || this.editor.setContent(value)
    }
  },
  beforeUnmount() {
    if (this.destroy && this.editor && this.editor.destroy) {
      // 原则上每次离开当前路由时，都应该销毁实例
      // 原因： 每次生成， 均会向DOM写入一个带有固定id的实例，存在重合的可能性。
      this.editor.destroy()
      this.editor = null
    }
  },
  // v-model语法糖实现
  watch: {
    modelValue: {
      handler(value) {
        if (this.editor) {
          this._setContent(value)
          // 响应事件， 用于获取纯文本方法，父组件通过监听puretxt获取当前编辑器的纯文本
        } else {
          this._beforeInitEditor(value)
        }
      },
      immediate: true
    }
  }
}
</script>
<style scoped>
.editor {
  width: 100%;
}
/deep/ .edui-editor {
  border: 1px solid #ececec !important;
}
</style>
