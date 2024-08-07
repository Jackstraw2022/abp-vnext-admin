<template>
  <Dropdown placement="bottomLeft" :overlayClassName="`${prefixCls}-dropdown-overlay`">
    <span :class="[prefixCls, `${prefixCls}--${theme}`]" class="flex">
      <img :class="`${prefixCls}__header`" :src="getUserInfo.avatar" />
      <span :class="`${prefixCls}__info hidden md:block`">
        <span :class="`${prefixCls}__name  `" class="truncate">
          {{ getUserInfo.realName }}
        </span>
      </span>
    </span>

    <template #overlay>
      <Menu @click="handleMenuClick">
        <MenuItem
          key="center"
          :text="t('AbpAccount.PersonalInfo')"
          icon="ant-design:profile-outlined"
        />
        <MenuItem
          key="setting"
          :text="t('AbpAccount.PersonalSettings')"
          icon="ant-design:setting-outlined"
        />
        <MenuItem
          key="security-logs"
          :text="t('AbpAuditLogging.SecurityLog')"
          icon="ant-design:security-scan-outlined"
        />
        <MenuItem
          key="sessions"
          :text="t('AbpIdentity.IdentitySessions')"
          icon="carbon:prompt-session"
        />
        <MenuDivider />
        <MenuItem
          v-if="getUseLockPage"
          key="lock"
          :text="t('layout.header.tooltipLock')"
          icon="ion:lock-closed-outline"
        />
        <MenuItem
          key="logout"
          :text="t('layout.header.dropdownItemLoginOut')"
          icon="ion:power-outline"
        />
      </Menu>
    </template>
  </Dropdown>
  <LockAction @register="register" />
  <SessionModal @register="registerSessionModal" />
  <SecurityLogsModal @register="registerSecurityLogsModal" />
</template>
<script lang="ts">
  // components
  import { Dropdown, Menu } from 'ant-design-vue';
  import type { MenuInfo } from 'ant-design-vue/lib/menu/src/interface';

  import { defineComponent, computed } from 'vue';

  import { DOC_URL } from '/@/settings/siteSetting';

  import { useUserStore } from '/@/store/modules/user';
  import { useAbpStoreWithOut } from '/@/store/modules/abp';
  import { useHeaderSetting } from '/@/hooks/setting/useHeaderSetting';
  import { useI18n } from '/@/hooks/web/useI18n';
  import { useDesign } from '/@/hooks/web/useDesign';
  import { useModal } from '/@/components/Modal';
  import { useGo } from '/@/hooks/web/usePage';

  import headerImg from '/@/assets/images/header.jpg';
  import { propTypes } from '/@/utils/propTypes';
  import { openWindow } from '/@/utils';

  import { createAsyncComponent } from '/@/utils/factory/createAsyncComponent';

  type MenuEvent = 'logout' | 'doc' | 'lock' | 'setting' | 'center' | 'sessions' | 'security-logs';

  export default defineComponent({
    name: 'UserDropdown',
    components: {
      Dropdown,
      Menu,
      MenuItem: createAsyncComponent(() => import('./DropMenuItem.vue')),
      MenuDivider: Menu.Divider,
      LockAction: createAsyncComponent(() => import('../lock/LockModal.vue')),
      SessionModal: createAsyncComponent(
        () => import('/@/views/account/sessions/index.vue'),
      ),
      SecurityLogsModal: createAsyncComponent(
        () => import('/@/views/account/security-logs/index.vue'),
      ),
    },
    props: {
      theme: propTypes.oneOf(['dark', 'light']),
    },
    setup() {
      const { prefixCls } = useDesign('header-user-dropdown');
      const { t } = useI18n();
      const go = useGo();
      const { getShowDoc, getUseLockPage } = useHeaderSetting();
      const userStore = useUserStore();
      const abpStore = useAbpStoreWithOut();

      const getUserInfo = computed(() => {
        const { currentTenant } = abpStore.getApplication;
        const { avatar, desc, realName, username } = userStore.getUserInfo || {};
        let userName = realName ?? username;
        if (currentTenant.name) {
          userName = `${currentTenant.name}/${userName}`;
        }
        return {
          realName: userName,
          avatar: avatar || headerImg,
          desc,
        };
      });

      const [register, { openModal }] = useModal();
      const [registerSessionModal, { openModal: openSessionModal }] = useModal();
      const [registerSecurityLogsModal, { openModal: openSecurityLogsModal }] = useModal();

      function handleLock() {
        openModal(true);
      }

      //  login out
      function handleLoginOut() {
        userStore.confirmLoginOut();
      }

      // open doc
      function openDoc() {
        openWindow(DOC_URL);
      }

      function handleMenuClick(e: MenuInfo) {
        switch (e.key as MenuEvent) {
          case 'logout':
            handleLoginOut();
            break;
          case 'doc':
            openDoc();
            break;
          case 'lock':
            handleLock();
            break;
          case 'setting':
            go('/account/settings');
            break;
          case 'center':
            go('/account/center');
            break;
          case 'sessions':
           openSessionModal(true, {});
            break;
          case 'security-logs':
            openSecurityLogsModal(true, {});
            break;
        }
      }

      return {
        prefixCls,
        t,
        getUserInfo,
        handleMenuClick,
        getShowDoc,
        register,
        getUseLockPage,
        registerSessionModal,
        registerSecurityLogsModal,
      };
    },
  });
</script>
<style lang="less">
  @prefix-cls: ~'@{namespace}-header-user-dropdown';

  .@{prefix-cls} {
    height: @header-height;
    padding: 0 0 0 10px;
    padding-right: 10px;
    overflow: hidden;
    font-size: 12px;
    cursor: pointer;
    align-items: center;

    img {
      width: 24px;
      height: 24px;
      margin-right: 12px;
    }

    &__header {
      border-radius: 50%;
    }

    &__name {
      font-size: 14px;
    }

    &--dark {
      &:hover {
        background-color: @header-dark-bg-hover-color;
      }
    }

    &--light {
      &:hover {
        background-color: @header-light-bg-hover-color;
      }

      .@{prefix-cls}__name {
        color: @text-color-base;
      }

      .@{prefix-cls}__desc {
        color: @header-light-desc-color;
      }
    }

    &-dropdown-overlay {
      .ant-dropdown-menu-item {
        min-width: 160px;
      }
    }
  }
</style>
