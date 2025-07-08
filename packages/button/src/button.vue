<template>
  <button
    class="el-button"
    @click="handleClick"
    :disabled="buttonDisabled || loading"
    :autofocus="autofocus"
    :type="nativeType"
    :class="[
      type ? 'el-button--' + type : '',
      buttonSize ? 'el-button--' + buttonSize : '',
      {
        'is-disabled': buttonDisabled,
        'is-loading': loading,
        'is-plain': plain,
        'is-round': round,
        'is-circle': circle
      }
    ]"
  >
    <!-- loading 时会隐藏掉 icon，只显示 loading -->
    <i class="el-icon-loading" v-if="loading"></i>
    <i :class="icon" v-if="icon && !loading"></i>
    <!-- 为何还要 v-if="$slots.default" 判断？ -->
    <span v-if="$slots.default"><slot></slot></span>
  </button>
</template>
<script>
  export default {
    name: 'ElButton',

    inject: {
      elForm: {
        default: ''
      },
      elFormItem: {
        default: ''
      }
    },

    props: {
      type: {
        type: String,
        default: 'default'
      },
      size: String,
      icon: {
        type: String,
        default: ''
      },
      nativeType: {
        type: String,
        default: 'button'
      },
      loading: Boolean,
      disabled: Boolean,
      plain: Boolean,
      autofocus: Boolean,
      round: Boolean,
      circle: Boolean
    },

    computed: {
      _elFormItemSize() {
        return (this.elFormItem || {}).elFormItemSize;
      },
      buttonSize() {
        return this.size || this._elFormItemSize || (this.$ELEMENT || {}).size;
      },
      /**
       * 计算按钮是否禁用
       * 优先级：
       * 1. 如果显式设置了 disabled prop，则使用 prop 的值
       * 2. 否则使用 elForm.disabled 的值（表单整体禁用状态）
       * @returns {Boolean} 按钮的禁用状态
       */
      buttonDisabled() {
        // 不能单纯使用 this.disabled 进行判断，因为有默认值得存在，单纯使用 this.disabled 无法区分是用户没传，还是传递了 false
        // 那么如何区分 “未传递” 与 “传递了 false”？用 propsData 判断！
        // this.$options.propsData 记录了父组件实际传递的 props。
        // this.$options.propsData.hasOwnProperty('disabled') 为 true，说明父组件明确传递了 disabled（无论 true 还是 false）。
        // 否则，说明父组件没有传递 disabled，此时才需要从 elForm 中获取 disabled 状态。
        return this.$options.propsData.hasOwnProperty('disabled') ? this.disabled : (this.elForm || {}).disabled;
      }
    },

    methods: {
      handleClick(evt) {
        this.$emit('click', evt);
      }
    }
  };
</script>
