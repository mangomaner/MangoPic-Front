<template>
  <div id="basicLayout">
    <a-layout style="min-height: 100vh">
      <a-layout-header class="header">
        <GlobalHeader />
      </a-layout-header>

      <a-layout>
        <GlobalSider class="sider" />
        <a-layout-content class="content">
          <router-view />
        </a-layout-content>
      </a-layout>

      <a-layout-footer class="footer">
        <a href="https://space.bilibili.com/521182571" style="color: #474747">@Mango Pic</a>
      </a-layout-footer>
    </a-layout>
  </div>
</template>

<script setup lang="ts">

import GlobalHeader from '@/components/GlobalHeader.vue'
import GlobalSider from '@/components/GlobalSider.vue'
</script>

<style scoped>
#basicLayout .header{
  background-color: white;
  color: unset;
  margin-bottom: 16px;
  padding-inline: 20px;
}

#basicLayout .content{
  padding: 20px;
  background-color: #fefefe;

}

#basicLayout .sider {
  background: #fff;
  padding-top: 20px;
  border-right: 0.5px solid #eee;
}

#basicLayout .footer{
  background-color: #efefef;
  bottom: 0;
  left: 0;
  right: 0;
  text-align: center;
}
</style>
