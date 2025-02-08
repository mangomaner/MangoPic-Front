<template>
  <div id="globalHeader">
    <a-row :wrap="false">
      <a-col flex="250px">
        <router-link to="/">
          <div class="titleBar">
            <img class="logo" src="../assets/logo.svg" alt="logo">
            <div class="title">Mango Pic</div>
          </div>
        </router-link>
      </a-col>
      <a-col flex="auto">
        <a-menu v-model:selectedKeys="current" mode="horizontal" :items="items" @click="doMenuClick"/>
      </a-col>
      <a-col flex="100px">
        <div class="user-login-status">
          <div v-if="loginUserStore.loginUser.id">
            {{loginUserStore.loginUser.username ?? 'unknown'}}
          </div>
          <div v-else>
            <a-button type="primary" href="/user/login">登录</a-button>
          </div>

        </div>
      </a-col>
    </a-row>
  </div>
  </template>
<script lang="ts" setup>
import { h, ref } from 'vue';
import { MailOutlined, AppstoreOutlined, SettingOutlined } from '@ant-design/icons-vue';
import { MenuProps } from 'ant-design-vue';
import { useRouter } from 'vue-router'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'

const loginUserStore = useLoginUserStore();

const items = ref<MenuProps['items']>([
  {
    key: '/',
    icon: () => h(MailOutlined),
    label: '主页',
    title: '主页',
  },
  {
    key: '/about',
    icon: () => h(AppstoreOutlined),
    label: '关于',
    title: '关于',
  },
  {
    key: 'sub1',
    icon: () => h(SettingOutlined),
    label: '其他',
    title: '其他',
    children: [
      {
        type: 'group',
        label: 'Item 1',
        children: [
          {
            label: 'Option 1',
            key: 'setting:1',
          },
          {
            label: 'Option 2',
            key: 'setting:2',
          },
        ],
      },
      {
        type: 'group',
        label: 'Item 2',
        children: [
          {
            label: 'Option 3',
            key: 'setting:3',
          },
          {
            label: 'Option 4',
            key: 'setting:4',
          },
        ],
      },
    ],
  },
  {
    key: 'alipay',
    label: h('a', { href: 'https://antdv.com', target: '_blank' }, 'Link'),
    title: 'Link',
  },
]);

const router = useRouter();

const doMenuClick = ({key}) => {
  router.push({
    path: key
  })
}

const current = ref<string[]>([]);
router.afterEach((to, from, next) => {
  current.value = [to.path];
})

</script>

<style scoped>
#globalHeader .titleBar{
  display: flex;
  align-items: center;
}

.title{
  color: #2c2c2c;
  font-size: 18px;
  margin-left: 16px;
}

.logo{
  height: 48px;
}

</style>

