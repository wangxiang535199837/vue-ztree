<template>
  <div :id="treeId" class="ztree">
  </div>
</template>
<script type="text/ecmascript-6">
  var index = 0

  export default {
    props: {
      // ztree 原生配置 参考 http://www.treejs.cn/v3/api.php
      setting: {
        type: Object,
        default: function () {
          return {}
        }
      },
      data: {
        type: Array,
        default: function () {
          return []
        }
      }
    },

    data: function () {
      return {
        treeId: 'ztree-' + index++,
        currentSelectedNodeId: ''
      }
    },

    mounted() {
      mountZtreeCallback(this.setting, this)
      const setting = this.setting
      if (setting.once) {
        const dataFilter = setting.once.dataFilter
        //不删除会在ajax调用过程中调用
        delete setting.once.dataFilter
        $.ajax(setting.once).done((data) => {
          $.fn.zTree.init($(this.$el), this.setting || {}, dataFilter(data) || null)
          //再在配置项里显示
          setting.once.dataFilter = dataFilter
        })
      } else {
        $.fn.zTree.init($(this.$el), this.setting || {}, this.data || null)
      }
    },

    methods: {
      /***
       * 调用ztree的原生方法
       * @param actionName 必选 需要调用的原生ztree方法名称
       * @param args 可选参数 调用方法时传入的参数
       * @returns 调用ztree原生方法的返回值
       */
      action(actionName, ...args) {
        var treeObj = $.fn.zTree.getZTreeObj(this.treeId)
        return treeObj[actionName](...args)
      },
      refresh: function (toMerge) {
        var treeObj = $.fn.zTree.getZTreeObj(this.treeId), setting = treeObj.setting,
          $dom = $('#' + setting.treeId)
        _.merge(setting, toMerge)
        $.fn.zTree.init($dom, setting, treeObj.getNodes())
      }
    }
  }

  function mountZtreeCallback (setting, vm) {
    const hooks = ['beforeAsync', 'beforeCheck', 'beforeClick', 'beforeCollapse', 'beforeDblClick', 'beforeDrag',
      'beforeDragOpen', 'beforeDrop', 'beforeEditName', 'beforeExpand', 'beforeMouseDown', 'beforeMouseUp',
      'beforeRemove', 'beforeRename', 'beforeRightClick', 'onAsyncError', 'onAsyncSuccess', 'onCheck', 'onClick',
      'onCollapse', 'onDblClick', 'onDrag', 'onDragMove', 'onDrop', 'onExpand', 'onMouseDown', 'onMouseUp',
      'onNodeCreated', 'onRemove', 'onRename', 'onRightClick']
    setting.callback = {}
    hooks.forEach(function (evtName) {
      setting.callback[evtName] = function () {
        const args = _.slice(arguments)
        const store = {}
        args.unshift(evtName)
        args.push(store)
        vm.$emit.apply(vm, args)
        return store.cancel === undefined ? true : !store.cancel
      }
    })
  }
</script>
