<template>
  <el-container :class="['my-layout', { 'is-app': isApp }, 'font-sans']">
    <div class="app-shadow" v-if="isApp && isOpen && !isHidden" @click="handleCloseAside"></div>
    <el-header :height="height" class="shadow-md" v-if="!HasAsideAll">
      <slot name="header"></slot>
    </el-header>
    <el-container class="my-content">
      <template v-if="hasAside">
        <!-- has aside -->
        <el-aside :class="['my-aside', { 'is-collapse': isCollapse }, { 'is-hidden': isHidden }, { 'is-open': isOpen }]" :width="width">
          <slot name="aside"></slot>
        </el-aside>
        <el-container>
          <el-header :height="height" class="shadow-md" v-if="HasAsideAll">
            <slot name="header"></slot>
          </el-header>
          <el-main class="m-2.5" style="--el-main-padding: 0">
            <slot></slot>
          </el-main>
        </el-container>
      </template>
      <template v-else>
        <el-main class="m-2.5" style="--el-main-padding: 0">
          <slot></slot>
        </el-main>
      </template>
    </el-container>
  </el-container>
</template>
<script lang="ts">
import { ref, computed } from '@vue/reactivity'
import { onMounted } from '@vue/runtime-core'
import { debounce } from 'lodash'
import { defineComponent } from 'vue'
import { useStore } from '../../store'

export default defineComponent({
  props: {
    height: {
      type: String,
      default: '60px'
    },
    hasAside: {
      type: Boolean,
      default: true
    },
    HasAsideAll: {
      type: Boolean,
      default: false
    }
  },
  setup() {
    const width = ref<string>('200px')
    const isCollapse = computed(() => store.state.app.isCollapse)
    const isHidden = computed(() => store.state.app.isHidden)
    const isOpen = computed(() => store.state.app.isOpen)
    const store = useStore()
    const CollapseWidth = computed(() => store.state.app.CollapseWidth)
    const HiddenWidth = computed(() => store.state.app.HiddenWidth)
    const isApp = computed(() => store.state.app.isApp)
    const CollapseMenuWidth = computed(() => `${store.state.app.CollapseMenuWidth}px`)
    const AsideBgColor = computed(() => store.state.color.menu.MenuBackColor)
    /**
     * @description: 用来控制窗口容器的resize ，通过vuex控制状态，自动 设置 class 样式
     * @param {*} void
     * @return {*}
     * @author: YaoYue
     */
    const handleResize = (): void => {
      let isCollapse = false,
        isHidden = false
      if (window.innerWidth > CollapseWidth.value) {
        isCollapse = false
        isHidden = false
        store.commit('app/setTypeBySizeAndKey', { key: 'isWeb', value: true })
      }
      if (window.innerWidth < CollapseWidth.value && window.innerWidth > HiddenWidth.value) {
        isCollapse = true
        isHidden = false
        store.commit('app/setTypeBySizeAndKey', { key: 'isPad', value: true })
      }
      if (window.innerWidth < HiddenWidth.value) {
        isCollapse = false
        isHidden = true
        store.commit('app/setTypeBySizeAndKey', { key: 'isApp', value: true })
      }
      store.commit('app/setIsHidden', isHidden)
      store.commit('app/setIsCollapse', isCollapse)
    }
    const handleOpen = (): void => {
      store.commit('app/setIsOpen', isHidden.value && !isOpen.value)
    }
    const handleCloseAside = (): void => {
      store.commit('app/setIsOpen', false)
    }
    // 初始化 isCollapse isHidden Status
    handleResize()
    onMounted(() => {
      // 监听 窗口折叠
      window.addEventListener('resize', debounce(handleResize, 150))
    })

    return {
      width,
      isCollapse,
      isHidden,
      isOpen,
      isApp,
      handleOpen,
      handleCloseAside,
      CollapseMenuWidth,
      AsideBgColor
    }
  }
})
</script>
<style lang="scss" scoped>
$AsideBgColor: v-bind(AsideBgColor);
$AppShadowBackColor: rgba(0, 0, 0, 0.25);
$CollapseWidth: v-bind(CollapseMenuWidth);

.my-layout {
  height: 100%;
  position: relative;
  --el-header-padding: 0;

  &.is-app {
    .is-open {
      position: absolute;
      height: 100%;
      left: 0;
      top: 0;
      z-index: 20;
    }
  }
  .app-shadow {
    position: absolute;
    left: 0;
    top: 0;
    right: 0;
    bottom: 0;
    background-color: $AppShadowBackColor;
    z-index: 10;
  }
  .my-content {
    flex: auto;
    .my-aside {
      background-color: $AsideBgColor;
      transition: width 0.28s ease-in-out;
      &.is-collapse {
        width: $CollapseWidth !important;
      }
      &.is-hidden {
        display: none;
      }
    }
  }
}
</style>
