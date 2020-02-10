<template>
  <div>
    <nuxt />
  </div>
</template>

<script lang="ts">
import { createComponent, ref } from '@vue/composition-api'

interface Item {
  icon: string
  title: string
  to: string
}

const useMenus = () => {
  const clipped = ref<boolean>(false)
  const drawer = ref<boolean>(false)
  const fixed = ref<boolean>(false)
  const items = ref<Item[]>([
    {
      icon: 'mdi-apps',
      title: 'Welcome',
      to: '/'
    },
    {
      icon: 'mdi-chart-bubble',
      title: 'Inspire',
      to: '/inspire'
    }
  ])
  const miniVariant = ref<boolean>(false)
  const right = ref<boolean>(false)
  const rightDrawer = ref<boolean>(false)

  return {
    clipped,
    drawer,
    fixed,
    items,
    miniVariant,
    right,
    rightDrawer
  }
}

export default createComponent({
  setup() {
    const menus = useMenus()
    const title = ref<string>('Vuetify.js')

    return {
      ...menus,
      title
    }
  }
})
</script>

<style>
html {
  font-family: 'Source Sans Pro', -apple-system, BlinkMacSystemFont, 'Segoe UI',
    Roboto, 'Helvetica Neue', Arial, sans-serif;
  font-size: 16px;
  word-spacing: 1px;
  -ms-text-size-adjust: 100%;
  -webkit-text-size-adjust: 100%;
  -moz-osx-font-smoothing: grayscale;
  -webkit-font-smoothing: antialiased;
  box-sizing: border-box;
}

*,
*:before,
*:after {
  box-sizing: border-box;
  margin: 0;
}

.button--green {
  display: inline-block;
  border-radius: 4px;
  border: 1px solid #3b8070;
  color: #3b8070;
  text-decoration: none;
  padding: 10px 30px;
}

.button--green:hover {
  color: #fff;
  background-color: #3b8070;
}

.button--grey {
  display: inline-block;
  border-radius: 4px;
  border: 1px solid #35495e;
  color: #35495e;
  text-decoration: none;
  padding: 10px 30px;
  margin-left: 15px;
}

.button--grey:hover {
  color: #fff;
  background-color: #35495e;
}
</style>
