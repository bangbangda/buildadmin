<template>
    <div class="nav-menus" :class="configStore.layout.layoutMode">
        <!-- 需要重启 Vite 热更新服务警告 -->
        <el-popover
            ref="reloadHotServerPopover"
            @show="onCurrentNavMenu(true, 'reloadHotServer')"
            @hide="onCurrentNavMenu(false, 'reloadHotServer')"
            :width="360"
            v-if="hotUpdateState.dirtyFile"
        >
            <div>
                <div class="el-popover__title">{{ t('vite.Reload hot server title') }}</div>
                <div class="reload-hot-server-content">
                    <p>
                        <span>{{ t('vite.Reload hot server tips 1') }}</span>
                        <span>【{{ t(`vite.Close type ${hotUpdateState.closeType}`) }}】</span>
                        <span>{{ t('vite.Reload hot server tips 2') }}</span>
                    </p>
                    <p>{{ t('vite.Reload hot server tips 3') }}</p>
                    <div class="reload-hot-server-buttons">
                        <el-button @click="onHotServerOpt('cancel')">{{ t('vite.Later') }}</el-button>
                        <el-button @click="onHotServerOpt('reload')" type="primary">{{ t('vite.Restart hot update') }}</el-button>
                    </div>
                </div>
            </div>
            <template #reference>
                <div class="nav-menu-item" :class="state.currentNavMenu == 'reloadHotServer' ? 'hover' : ''">
                    <Icon color="var(--el-color-danger)" class="nav-menu-icon" name="el-icon-Warning" size="18" />
                </div>
            </template>
        </el-popover>

        <!-- 站点主页 -->
        <router-link class="h100" target="_blank" :title="t('Home')" to="/">
            <div class="nav-menu-item">
                <Icon :color="configStore.getColorVal('headerBarTabColor')" class="nav-menu-icon" name="el-icon-Monitor" size="18" />
            </div>
        </router-link>

        <!-- 语言切换 -->
        <el-dropdown
            @visible-change="onCurrentNavMenu($event, 'lang')"
            class="h100"
            size="large"
            :hide-timeout="50"
            placement="bottom"
            trigger="click"
            :hide-on-click="true"
        >
            <div class="nav-menu-item pt2" :class="state.currentNavMenu == 'lang' ? 'hover' : ''">
                <Icon :color="configStore.getColorVal('headerBarTabColor')" class="nav-menu-icon" name="local-lang" size="18" />
            </div>
            <template #dropdown>
                <el-dropdown-menu class="dropdown-menu-box">
                    <el-dropdown-item v-for="item in configStore.lang.langArray" :key="item.name" @click="editDefaultLang(item.name)">
                        {{ item.value }}
                    </el-dropdown-item>
                </el-dropdown-menu>
            </template>
        </el-dropdown>

        <!-- 全屏切换 -->
        <div @click="onFullScreen" class="nav-menu-item" :class="state.isFullScreen ? 'hover' : ''">
            <Icon
                :color="configStore.getColorVal('headerBarTabColor')"
                class="nav-menu-icon"
                v-if="state.isFullScreen"
                name="local-full-screen-cancel"
                size="18"
            />
            <Icon :color="configStore.getColorVal('headerBarTabColor')" class="nav-menu-icon" v-else name="el-icon-FullScreen" size="18" />
        </div>

        <!-- 终端 - 仅超管 -->
        <div v-if="adminInfo.super" @click="terminal.toggle()" class="nav-menu-item pt2">
            <el-badge :is-dot="terminal.state.showDot">
                <Icon :color="configStore.getColorVal('headerBarTabColor')" class="nav-menu-icon" name="local-terminal" size="26" />
            </el-badge>
        </div>

        <!-- 清理缓存 - 仅超管 -->
        <el-dropdown
            v-if="adminInfo.super"
            @visible-change="onCurrentNavMenu($event, 'clear')"
            class="h100"
            size="large"
            :hide-timeout="50"
            placement="bottom"
            trigger="click"
            :hide-on-click="true"
        >
            <div class="nav-menu-item" :class="state.currentNavMenu == 'clear' ? 'hover' : ''">
                <Icon :color="configStore.getColorVal('headerBarTabColor')" class="nav-menu-icon" name="el-icon-Delete" size="18" />
            </div>
            <template #dropdown>
                <el-dropdown-menu class="dropdown-menu-box">
                    <el-dropdown-item @click="onClearCache('tp')">{{ t('utils.Clean up system cache') }}</el-dropdown-item>
                    <el-dropdown-item @click="onClearCache('storage')">{{ t('utils.Clean up browser cache') }}</el-dropdown-item>
                    <el-dropdown-item @click="onClearCache('all')" divided>{{ t('utils.Clean up all cache') }}</el-dropdown-item>
                </el-dropdown-menu>
            </template>
        </el-dropdown>

        <!-- 管理员信息 -->
        <el-popover
            v-if="siteConfig.userInitialize"
            @show="onCurrentNavMenu(true, 'adminInfo')"
            @hide="onCurrentNavMenu(false, 'adminInfo')"
            placement="bottom-end"
            :hide-after="0"
            :width="260"
            trigger="click"
            popper-class="admin-info-box"
            v-model:visible="state.showAdminInfoPopover"
        >
            <template #reference>
                <div class="admin-info" :class="state.currentNavMenu == 'adminInfo' ? 'hover' : ''">
                    <el-avatar :size="25" :src="fullUrl(adminInfo.avatar)"></el-avatar>
                    <div class="admin-name">{{ adminInfo.nickname }}</div>
                </div>
            </template>
            <div>
                <div class="admin-info-base">
                    <el-avatar :size="70" :src="fullUrl(adminInfo.avatar)"></el-avatar>
                    <div class="admin-info-other">
                        <div class="admin-info-name">{{ adminInfo.nickname }}</div>
                        <div class="admin-info-lasttime">{{ adminInfo.last_login_time }}</div>
                    </div>
                </div>
                <div class="admin-info-footer">
                    <el-button @click="onAdminInfo" type="primary" plain>{{ t('layouts.Profile') }}</el-button>
                    <el-button @click="onLogout" type="danger" plain>{{ t('layouts.Logout') }}</el-button>
                </div>
            </div>
        </el-popover>

        <!-- 配置 -->
        <div @click="configStore.setLayout('showDrawer', true)" class="nav-menu-item">
            <Icon :color="configStore.getColorVal('headerBarTabColor')" class="nav-menu-icon" name="fa fa-cogs" size="18" />
        </div>

        <Config />
        <Terminal />
    </div>
</template>

<script lang="ts" setup>
import { ElMessage } from 'element-plus'
import screenfull from 'screenfull'
import { reactive, useTemplateRef } from 'vue'
import { useI18n } from 'vue-i18n'
import Config from './config.vue'
import { logout } from '/@/api/backend/index'
import { postClearCache } from '/@/api/common'
import Terminal from '/@/components/terminal/index.vue'
import { editDefaultLang } from '/@/lang'
import router from '/@/router'
import { useAdminInfo } from '/@/stores/adminInfo'
import { useConfig } from '/@/stores/config'
import { ADMIN_INFO, BA_ACCOUNT } from '/@/stores/constant/cacheKey'
import { useSiteConfig } from '/@/stores/siteConfig'
import { useTerminal } from '/@/stores/terminal'
import { fullUrl } from '/@/utils/common'
import { routePush } from '/@/utils/router'
import { Local, Session } from '/@/utils/storage'
import { hotUpdateState, reloadServer } from '/@/utils/vite'

const { t } = useI18n()

const adminInfo = useAdminInfo()
const configStore = useConfig()
const terminal = useTerminal()
const siteConfig = useSiteConfig()
const reloadHotServerPopover = useTemplateRef('reloadHotServerPopover')

const state = reactive({
    isFullScreen: false,
    currentNavMenu: '',
    showLayoutDrawer: false,
    showAdminInfoPopover: false,
})

const onCurrentNavMenu = (status: boolean, name: string) => {
    state.currentNavMenu = status ? name : ''
}

const onHotServerOpt = (opt: 'reload' | 'cancel') => {
    if (opt == 'cancel') {
        reloadHotServerPopover.value?.hide()
    } else {
        reloadServer('manual')
    }
}

const onFullScreen = () => {
    if (!screenfull.isEnabled) {
        ElMessage.warning(t('layouts.Full screen is not supported'))
        return false
    }
    screenfull.toggle()
    screenfull.onchange(() => {
        state.isFullScreen = screenfull.isFullscreen
    })
}

const onAdminInfo = () => {
    state.showAdminInfoPopover = false
    routePush({ name: 'routine/adminInfo' })
}

const onLogout = () => {
    logout().then(() => {
        Local.remove(ADMIN_INFO)
        router.go(0)
    })
}

const onClearCache = (type: string) => {
    if (type == 'storage' || type == 'all') {
        const adminInfo = Local.get(ADMIN_INFO)
        const baAccount = Local.get(BA_ACCOUNT)
        Session.clear()
        Local.clear()
        Local.set(ADMIN_INFO, adminInfo)
        Local.set(BA_ACCOUNT, baAccount)
        if (type == 'storage') return
    }
    postClearCache(type).then(() => {})
}
</script>

<style scoped lang="scss">
.nav-menus.Default {
    border-radius: var(--el-border-radius-base);
    box-shadow: var(--el-box-shadow-light);
}
.reload-hot-server-content {
    font-size: var(--el-font-size-small);
    p {
        margin-bottom: 6px;
    }
    .reload-hot-server-buttons {
        display: flex;
        justify-content: flex-end;
    }
}
.nav-menus {
    display: flex;
    align-items: center;
    height: 100%;
    margin-left: auto;
    background-color: v-bind('configStore.getColorVal("headerBarBackground")');
    .nav-menu-item {
        height: 100%;
        width: 40px;
        display: flex;
        align-items: center;
        justify-content: center;
        cursor: pointer;
        .nav-menu-icon {
            box-sizing: content-box;
            color: v-bind('configStore.getColorVal("headerBarTabColor")');
        }
        &:hover {
            .icon {
                animation: twinkle 0.3s ease-in-out;
            }
        }
    }
    .admin-info {
        display: flex;
        height: 100%;
        padding: 0 10px;
        align-items: center;
        cursor: pointer;
        user-select: none;
        color: v-bind('configStore.getColorVal("headerBarTabColor")');
    }
    .admin-name {
        padding-left: 6px;
        white-space: nowrap;
    }
    .nav-menu-item:hover,
    .admin-info:hover,
    .nav-menu-item.hover,
    .admin-info.hover {
        background: v-bind('configStore.getColorVal("headerBarHoverBackground")');
    }
}
.dropdown-menu-box :deep(.el-dropdown-menu__item) {
    justify-content: center;
}
.admin-info-base {
    display: flex;
    justify-content: center;
    flex-wrap: wrap;
    padding-top: 10px;
    .admin-info-other {
        display: block;
        width: 100%;
        text-align: center;
        padding: 10px 0;
        .admin-info-name {
            font-size: var(--el-font-size-large);
        }
    }
}
.admin-info-footer {
    padding: 10px 0;
    margin: 0 -12px -12px -12px;
    display: flex;
    justify-content: space-around;
}
.pt2 {
    padding-top: 2px;
}

@keyframes twinkle {
    0% {
        transform: scale(0);
    }
    80% {
        transform: scale(1.2);
    }
    100% {
        transform: scale(1);
    }
}
</style>
