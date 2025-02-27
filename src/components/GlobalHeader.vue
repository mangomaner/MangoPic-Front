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
          <div v-if="loginUserStore.loginUser.id" style="cursor: pointer">
            <a-dropdown>
              <ASpace>
                <a-avatar :src="loginUserStore.loginUser.userAvatar" />
                {{ loginUserStore.loginUser.userName ?? "unKnown" }}
              </ASpace>
              <template #overlay>
                <a-menu>
                  <a-menu-item>
                    <router-link to="/my_space">
                      <UserOutlined />
                      我的空间
                    </router-link>
                  </a-menu-item>

                  <a-menu-item @click="doLogout">
                    <LogoutOutlined />
                    退出登录
                  </a-menu-item>
                </a-menu>
              </template>
            </a-dropdown>
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
import { computed, h, ref } from 'vue'
import { MailOutlined, AppstoreOutlined, SettingOutlined, HomeOutlined } from '@ant-design/icons-vue';
import { MenuProps, message } from 'ant-design-vue'
import { useRouter } from 'vue-router'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'
import { userLogoutUsingPost } from '@/api/userController.ts'

const loginUserStore = useLoginUserStore();

// 菜单列表
const originItems = [
  {
    key: '/',
    icon: () => h(HomeOutlined),
    label: '主页',
    title: '主页',
  },
  {
    key: '/add_picture',
    label: '创建图片',
    title: '创建图片',
  },
  {
    key: '/admin/userManage',
    label: '用户管理',
    title: '用户管理',
  },
  {
    key: '/admin/pictureManage',
    label: '图片管理',
    title: '图片管理',
  },
  {
    key: '/admin/spaceManage',
    label: '空间管理',
    title: '空间管理',
  },
  {
    key: 'others',
    label: '关于',
    title: '关于',
  },
  // {
  //   key: 'others',
  //   label: h('a', { href: 'https://www.codefather.cn', target: '_blank' }, '编程导航'),
  //   title: '编程导航',
  // },
]

// 过滤菜单项
const filterMenus = (menus = [] as MenuProps['items']) => {
  return menus?.filter((menu) => {
    if (menu.key.startsWith('/admin')) {
      const loginUser = loginUserStore.loginUser
      if (!loginUser || loginUser.userRole !== "admin") {
        return false
      }
    }
    return true
  })
}

// 展示在菜单的路由数组
const items = computed<MenuProps['items']>(() => filterMenus(originItems))


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

// 用户注销
const doLogout = async () => {
  const res = await userLogoutUsingPost()
  console.log(res)
  if (res.data.code === 0) {
    loginUserStore.setLoginUser({
      userName: '未登录',
    })
    message.success('退出登录成功')
    await router.push('/user/login')
  } else {
    message.error('退出登录失败，' + res.data.message)
  }
}


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

