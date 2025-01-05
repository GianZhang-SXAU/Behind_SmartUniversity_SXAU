<template>
  <div class="Home">
    <div class="header">
      <div class="header-left">
        <div class="toggleChunk" @click="ToggleCollapsed">
          <MenuUnfoldOutlined v-if="siderCollapsed"/>
          <MenuFoldOutlined v-else/>
        </div>
      </div>
      <div class="header-center">
        <p class="header-center-title">{{ title }}</p>
      </div>
      <div class="header-right">
        <a-avatar>
          <template #icon>
            <UserOutlined v-if="useState.$state.avatarurl == ''"/>
            <img :src="useState.$state.avatarurl"/>
          </template>
        </a-avatar>
        <p class="header-right-title">{{ useState.$state.name }}</p>
        <a-button type="link" @click="LoginOut">退出</a-button>
      </div>
    </div>
    <div class="content">
      <a-menu class="siderMenu" theme="dark" v-model:selectedKeys="selectedKeys" :openKeys="openKeysArr"
              :inline-collapsed="siderCollapsed" mode="inline">
        <template v-for="(v, k) in menuList" :key="k">
          <a-menu-item v-if="!Array.isArray(v)" @click="MenuItemClick(v)" :key="v.path">
            <component :is="v.meta.icon"></component>
            <span>{{ k }}</span>
          </a-menu-item>
          <a-sub-menu v-else :key="v">
            <template #title>
              <component :is="v[0].icon"></component>
              <span>
                                {{ k }}
                            </span>
            </template>
            <template v-for="(c) in v" :key="c.path">
              <a-menu-item @click="MenuItemClick(c)">
                <component :is="c.meta.icon"></component>
                <span>{{ c.meta.title }}</span>
              </a-menu-item>
            </template>
          </a-sub-menu>
        </template>
      </a-menu>
      <div class="content-content">
        <a-card class="contentHeader">
          <a-page-header :title="tarMenuTitle" @back="() => router.back()"/>
        </a-card>
        <a-card class="contentCard">
          <router-view></router-view>
        </a-card>
      </div>
    </div>

  </div>
</template>
<script setup>
import {
  UserOutlined,
  VideoCameraOutlined,
  UploadOutlined,
  MenuFoldOutlined,
  MenuUnfoldOutlined
} from '@ant-design/icons-vue';
import {useStore} from '@/store';

const router = useRouter();
const route = useRoute();
const title = import.meta.env.VITE_WEB_TITLE;
let siderCollapsed = ref(false);
const selectedKeys = ref();
const openKeysArr = ref([router.currentRoute.value.meta.parentTitle]);
const useState = useStore();
const menuList = ref([]);
const breadcrumbList = ref([]);
let tarMenuTitle = ref('');
const panes = reactive([]);
let activeKey = ref();

// 菜单点击
const MenuItemClick = (tarVal) => {
  const {meta, name} = tarVal;
  router.push({
    name: tarVal.name
  })
  activeKey.value = name;
  if (panes.some(i => i.key == name)) return;
  panes.push({
    title: meta.title, key: name,
  })
}
const LoginOut = async () => {
  let flag = await useState.LoginOut()
  if (flag) {
    router.push({
      name: 'Login'
    })
  }
}
const TabsEdit = (targetKey, action) => {
  if (action === 'remove') {
    if (panes.length == 1) return
    if (activeKey.value == panes[panes.length - 1].key) {
      activeKey.value = panes[panes.length - 2].key;
    } else if (activeKey.value == panes[0].key) {
      activeKey.value = panes[1].key;
    } else if (activeKey.value == targetKey) {
      activeKey.value = panes[panes.findIndex(i => i.key == targetKey) - 1].key;
    }
    router.push({
      name: activeKey.value
    })
    panes.splice(panes.findIndex(i => i.key == targetKey), 1);
  }
}
const TabsChange = (activeKey) => {
  let tarObj = panes.filter(i => i.key == activeKey)[0];
  router.push({
    name: tarObj.key
  })
}
onMounted(() => {
  const {meta, name} = router.currentRoute.value;
  panes.push({
    title: meta.title, key: name, closable: true
  })
  let childrenArr = router['options'].routes.filter(i => i.name == 'Home')[0].children;
  let newMenuList = childrenArr.concat(useState.routes).reduce((res, item) => {
    if (item.meta.parentTitle == undefined) {
      res[item.meta.title] = item
    } else {
      res[item.meta.parentTitle] ? res[item.meta.parentTitle].push(item) : res[item.meta.parentTitle] = [item]
    }

    return res;
  }, {});
  menuList.value = newMenuList;
})
watchEffect(() => {
  selectedKeys.value = [router.currentRoute.value.path]
  tarMenuTitle.value = router.currentRoute.value.meta.title;
  breadcrumbList.value = route.matched;
})
const ToggleCollapsed = () => {
  siderCollapsed.value = !siderCollapsed.value
};
</script>
<style lang="scss" scoped>
.Home {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  background: transparent;
  // background: #f0f2f5;

  .header {
    width: 100%;
    height: 80px;
    background: #001529;
    color: #fff;
    display: flex;
    align-items: center;
    justify-content: space-between;
    width: 100%;
    box-sizing: border-box;
    padding: 16px 12px;

    .header-left {
      display: flex;
      align-items: center;
      justify-content: center;
      color: #fff;

      .toggleChunk {
        margin-right: 10px;
        color: #fff;
      }

      :deep(.ant-breadcrumb) {

        .ant-breadcrumb-link,
        .ant-breadcrumb-separator {
          color: #fff !important;
        }
      }

      &-title {
        font-size: 20px;
        font-weight: 600;
      }

      &-secondTitle {
        font-size: 14px;
        margin-left: 12px;
        font-weight: 600;
      }
    }

    &-center {
      font-size: 26px;
      font-weight: bold;
    }

    .header-right {
      display: flex;
      align-items: center;

      &-dropdown {
        margin-left: 5px;


      }

      .ant-dropdown-link {
        color: #fff;
      }

      &-title {
        font-size: 14px;
        font-weight: 500;
        margin-left: 5px;
      }
    }
  }


  .content {
    max-width: 100%;
    height: calc(100% - 80px);
    display: flex;
    background: transparent;
    // background: #f0f2f5;
    z-index: 2;

    .siderMenu {
      max-width: 200px;
      height: 100%;
      overflow-y: auto;
      background: transparent;
      background: #001529;
      box-shadow: 0px 10px 25px 5px #0000000f;
    }


    &-breadcrumb {
      background: #fff;
      margin: 5px 0;

      :deep(.ant-card-body) {
        padding: 12px;
        display: flex;

        .toggleChunk {
          margin-right: 10px;
        }
      }

    }

    &-content {
      display: flex;
      flex-direction: column;
      width: 100%;
      height: calc(100%);
      padding: 12px 12px;
      overflow: hidden;
      box-sizing: border-box;

      background: transparent;
      background: url('../Login/img/主页背景.png');
      background-size: cover;
      box-shadow: 0px 10px 25px 5px #0000000f;

      .contentHeader {
        background: transparent;
        border: 0;

        :deep(.ant-card-body) {
          padding: 0;


          .ant-page-header {
            padding: 5px 0;

            .ant-page-header-back-button,
            .ant-page-header-heading-title {
              color: #Fff;
            }

          }
        }
      }

      .contentCard {
        width: 100%;
        height: 100%;
        // max-height: calc(100%);
        overflow-y: scroll;
        background: transparent;
        border: none;

        box-shadow: 0px 10px 25px 5px #0000000f;

      }

      .contentTabs {
        width: 100%;
        max-height: calc(100%);
        overflow-y: scroll;


        :deep(.ant-tabs-nav-add) {
          display: none;
        }

        :deep(.ant-tabs-content-holder) {
          padding: 0 12px;
        }
      }
    }
  }

  .bg-animation {
    position: absolute;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    z-index: 1;
    overflow: hidden;

    .top {
      position: absolute;
      max-height: 40rem;
      max-width: 40rem;
      height: 100%;
      width: 100%;
      border-radius: 50%;
      background-image: linear-gradient(80deg, #e0d3ef, rgb(192, 60, 165));
      filter: blur(40px);
      animation: bottom 30s infinite;
    }

    @keyframes bottom {

      0%,
      100% {
        top: -10rem;
      }

      80% {
        top: 50rem;
      }
    }

    .bottom {
      position: absolute;
      max-height: 25rem;
      max-width: 25rem;
      height: 100%;
      width: 100%;
      right: 0;
      border-radius: 50%;
      background-image: linear-gradient(80deg, rgb(192, 60, 165), #ebe9ee);
      filter: blur(2rem);
      animation: top 30s infinite;
    }

    @keyframes top {

      0%,
      100% {
        bottom: -10rem;
      }

      80% {
        bottom: 25rem;
      }
    }

    .left {
      position: absolute;
      max-height: 25rem;
      max-width: 25rem;
      height: 100%;
      width: 100%;
      left: 0;
      border-radius: 50%;
      background-image: linear-gradient(80deg, #ebe9ee, #a42322);
      filter: blur(6rem);
      animation: left 30s infinite;
    }

    @keyframes left {

      0%,
      100% {
        left: -10rem;
      }

      80% {
        left: 80rem;
      }
    }

    .right {
      position: absolute;
      max-height: 40rem;
      max-width: 40rem;
      height: 100%;
      width: 100%;
      right: 0;
      border-radius: 50%;
      background: linear-gradient(80deg, #a42322, #34278d);
      filter: blur(30px);
      animation: right 30s infinite;

    }

    @keyframes right {

      0%,
      100% {
        right: -10rem;

      }

      80% {
        right: 80rem;

      }
    }

  }
}
</style>