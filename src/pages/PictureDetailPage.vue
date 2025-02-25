<template>
  <a-row :gutter="[16, 16]" style="margin-top: -20px">
    <!-- 图片展示区 -->
    <a-col :sm="24" :md="16" :xl="18">
      <a-card style="text-align: center">
        <a-image
          ref="imageRef"
          style="max-height: 600px; object-fit: contain"
          :preview="{
            visible: previewVisible,
            onVisibleChange: handlePreviewVisibleChange,
          }"
          :src="picture.url"
        />
      </a-card>
    </a-col>
    <!-- 图片信息区 -->
    <a-col :sm="24" :md="8" :xl="6">
      <a-card title="图片信息">
        <a-descriptions :column="1">
          <a-descriptions-item label="作者">
            <a-space>
              <a-avatar :size="24" :src="picture.user?.userAvatar" />
              <div>{{ picture.user?.userName }}</div>
            </a-space>
          </a-descriptions-item>
          <a-descriptions-item label="名称">
            {{ picture.name ?? '未命名' }}
          </a-descriptions-item>
          <a-descriptions-item label="简介">
            {{ picture.introduction ?? '-' }}
          </a-descriptions-item>
          <a-descriptions-item label="分类">
            {{ picture.category ?? '默认' }}
          </a-descriptions-item>
          <a-descriptions-item label="标签">
            <a-tag v-for="tag in picture.tags" :key="tag">
              {{ tag }}
            </a-tag>
          </a-descriptions-item>
          <a-descriptions-item label="格式">
            {{ picture.picFormat ?? '-' }}
          </a-descriptions-item>
          <a-descriptions-item label="宽度">
            {{ picture.picWidth ?? '-' }}
          </a-descriptions-item>
          <a-descriptions-item label="高度">
            {{ picture.picHeight ?? '-' }}
          </a-descriptions-item>
          <a-descriptions-item label="宽高比">
            {{ picture.picScale ?? '-' }}
          </a-descriptions-item>
          <a-descriptions-item label="大小">
            {{ formatSize(picture.picSize) }}
          </a-descriptions-item>
        </a-descriptions>
        <a-space wrap>
          <a-button :icon="h(DownloadOutlined)" type="primary" @click="doDownload">
            下载
            <template #icon>
              <DownloadOutlined />
            </template>
          </a-button>
          <a-button type="primary" ghost @click="doShare">
            分享
            <template #icon>
              <share-alt-outlined />
            </template>
          </a-button>
          <a-button v-if="canEdit" :icon="h(EditOutlined)" type="default" @click="doEdit">
            编辑
            <template #icon>
              <EditOutlined />
            </template>
          </a-button>
          <a-button v-if="canEdit" :icon="h(DeleteOutlined)" danger @click="doDelete">
            删除
            <template #icon>
              <DeleteOutlined />
            </template>
          </a-button>
        </a-space>

      </a-card>
    </a-col>
  </a-row>

</template>

<script setup lang="ts">
import { deletePictureUsingPost, getPictureVoByIdUsingGet } from '@/api/pictureController.ts'
import { computed, onMounted, ref, nextTick, h } from 'vue'
import { message } from 'ant-design-vue'
import { downloadImage, formatSize } from '../utils'
import { ShareAltOutlined, EditOutlined, DeleteOutlined, DownloadOutlined } from '@ant-design/icons-vue'
import { useLoginUserStore } from '@/stores/useLoginUserStore.ts'
import router from '@/router'

const props = defineProps<{
  id: string | number
}>()
const picture = ref<API.PictureVO>({})
const imageRef = ref<HTMLElement | null>(null)
const previewVisible = ref(false)
let currentScale = 1 // 当前缩放比例
let transformOriginX = '50%' // 初始缩放中心点 X 轴位置
let transformOriginY = '50%' // 初始缩放中心点 Y 轴位置

// 获取图片详情
const fetchPictureDetail = async () => {
  try {
    const res = await getPictureVoByIdUsingGet({
      id: props.id,
    })
    if (res.data.code === 0 && res.data.data) {
      picture.value = res.data.data
    } else {
      message.error('获取图片详情失败，' + res.data.message)
    }
  } catch (e: any) {
    message.error('获取图片详情失败：' + e.message)
  }
}

onMounted(() => {
  fetchPictureDetail()
})

const loginUserStore = useLoginUserStore()

// 是否具有编辑权限
const canEdit = computed(() => {
  const loginUser = loginUserStore.loginUser;
  // 未登录不可编辑
  if (!loginUser.id) {
    return false
  }
  // 仅本人或管理员可编辑
  const user = picture.value.user || {}
  return loginUser.id === user.id || loginUser.userRole === 'admin'
})

// 编辑
const doEdit = () => {
  router.push({
    path: '/add_picture',
    query: {
      id: picture.value.id,
      spaceId: picture.value.spaceId
    }
  })
}

// 删除
const doDelete = async () => {
  const id = picture.value.id
  if (!id) {
    return
  }
  const res = await deletePictureUsingPost({ id })
  if (res.data.code === 0) {
    message.success('删除成功')
  } else {
    message.error('删除失败')
  }
  router.back();
}

// 处理下载
const doDownload = () => {
  downloadImage(picture.value.url)
}


// 预览可见性变化处理
const handlePreviewVisibleChange = (visible: boolean) => {
  previewVisible.value = visible

  if (visible) {
    // 预览打开时初始化缩放中心点
    nextTick(() => {
      const imgElement = document.querySelector('.ant-image-preview-img')
      const originPic = document.querySelector('.ant-image-preview-img')?.getBoundingClientRect();
      if (imgElement) {
        imgElement.addEventListener('mousemove', (event: MouseEvent) => {
          initZoomCenter(event, originPic)
        })

        imgElement.addEventListener('wheel', (event: WheelEvent) => {

          handleWheelZoom(event)
        })
      }
    })
  } else {
    // 预览关闭时重置缩放中心点和缩放比例
    // nextTick(() => {
    //   const imgElement = document.querySelector('.ant-image-preview-img')
    //   // if (imgElement) {
    //   //   imgElement.style.transformOrigin = '50% 50%'
    //   //   imgElement.style.transform = 'scale(1)'
    //   //   currentScale = 1
    //   //   transformOriginX = '50%'
    //   //   transformOriginY = '50%'
    //   // }
    // })
  }
}
let lastClientX = 0;
let lastClientY = 0;
// 初始化缩放中心点
const initZoomCenter = (event: MouseEvent, originPic: DOMRect) => {
  const imgElement = document.querySelector('.ant-image-preview-img')

  if (imgElement && event) {

    // 根据鼠标位置设置缩放中心点
    const rect = imgElement.getBoundingClientRect()
    //console.log(rect);
    if(Math.abs(event.clientX - lastClientX) > originPic.width / 100 && Math.abs(event.clientY - lastClientY) > originPic.height / 100) {
      const xPart = (event.clientX - rect.left) / rect.width;
      const yPart = (event.clientY - rect.top) / rect.height;

      transformOriginX = `${originPic.width * xPart}px`
      transformOriginY = `${originPic.height * yPart}px`
      //console.log('mX:', event.clientX, '\nmY:', event.clientY, '\ntX:', transformOriginX, '\ntY:', transformOriginY)
      lastClientX = event.clientX;
      lastClientY = event.clientY;
    }
  }
}

// 处理滚轮缩放
const handleWheelZoom = (event: WheelEvent) => {
  const imgElement = document.querySelector('.ant-image-preview-img')

  if (imgElement) {
    event.preventDefault() // 阻止默认行为

    // const delta = event.deltaY > 0 ? -0.5 : 0.5 // 判断滚轮方向
    // currentScale += delta
    //
    // // 限制缩放范围
    // if (currentScale < 0.5){
    //   currentScale = 0.5
    // }
    // if (currentScale > 2){
    //   currentScale = 2
    // }

    imgElement.style.transformOrigin = `${transformOriginX} ${transformOriginY}`
    //imgElement.style.transform = `scale(${currentScale})`
  }
}
</script>

<style scoped>
a-image :deep(.ant-image-preview-img){
  transition: transform 1s ease;
}
</style>
