<template>
  <el-menu
    active-text-color="#ffd04b"
    :background-color="variables.menuBg"
    :default-active="defaultActive"
    class="el-menu-vertical-demo"
    text-color="#fff"
    router
    unique-opened
    :collapse="!$store.getters.siderType"
  >
    <el-sub-menu :index="item.id" v-for="(item, index) in menusList" :key="item.id">
      <template #title>
        <el-icon>
          <component :is="iconList[index]"></component>
        </el-icon>
        <span>{{ item.authName }}</span>
      </template>
      <el-menu-item :index="'/' + it.path" v-for="it in item.children" :key="it.id" @click="savePath(it.path)">{{ $t(`menus.${it.path}`) }}</el-menu-item>
    </el-sub-menu>
  </el-menu>
</template>

<script setup>
import { menuList } from '@/api/menu'
import { ref } from 'vue'
import variables from '@/styles/variables.scss'

const defaultActive = ref(sessionStorage.getItem('path' || '/users'))
const savePath = (path) => {
  sessionStorage.setItem('path', `/${path}`)
}

const menusList = ref([])
const initMenuList = async () => {
  menusList.value = await menuList()
}
initMenuList()

// 小图标
const iconList = ref(['user', 'setting', 'shop', 'tickets', 'pie-chart'])
</script>

<style lang="scss" scoped></style>
