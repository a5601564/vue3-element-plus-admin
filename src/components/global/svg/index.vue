<!--
 * @Description: 本地svg图片使用组件
 * @Author: gumingchen
 * @Email: 1240235512@qq.com
 * @Date: 1995-08-30 05:20:00
 * @LastEditors: gumingchen
 * @LastEditTime: 1995-08-30 05:20:00
-->
<template>
  <svg
    aria-hidden="true"
    :class="iconClass"
    :width="size"
    :height="size"
    :color="color">
    <use :xlink:href="iconName" />
  </svg>
</template>
<!--
 * 使用:
 *   1.组件模版中使用
 *      [<g-svg
 *            name="icon"
 *            size="100"
 *            color="red"
 *            svgClass="svg"></g-svg>]
 * 注意:
 *    1.name      为svg图片名称，不需要扩展名 （必填）
 *    2.size      为svg高宽（默认1em）
 *    3.color     为svg颜色
 *    4.svgClass  为svg Class样式
-->
<script>
import { computed, defineComponent } from 'vue'

export default defineComponent({
  props: {
    // icon 名称 必传
    name: { type: String, required: true },
    // 自定义class name
    svgClass: { type: String, default: '' },
    // size 大小
    size: { type: String, default: '16px' },
    // color 颜色
    color: { type: String, default: '' }
  },
  setup(props) {
    /**
     * @description: 名称处理
     * @param {*}
     * @return {*}
     * @author: gumingchen
     */
    const iconName = computed(() => {
      const result = `#${ props.name }`
      return result
    })

    /**
     * @description: 样式处理
     * @param {*}
     * @return {*}
     * @author: gumingchen
     */
    const iconClass = computed(() => {
      let result
      if (props.svgClass) {
        result = 'g-svg-set ' + props.svgClass
      } else {
        if (props.size) {
          result = 'g-svg-set '
        } else {
          result = 'g-svg'
        }
      }
      return result
    })

    return {
      iconName,
      iconClass
    }
  }

})
</script>

<style lang="scss" scoped>
.g-svg {
  width: 1em;
  height: 1em;
  vertical-align: -0.15em;
  fill: currentColor;
  overflow: hidden;
}
.g-svg-set {
  vertical-align: -0.15em;
  fill: currentColor;
  overflow: hidden;
}
</style>
