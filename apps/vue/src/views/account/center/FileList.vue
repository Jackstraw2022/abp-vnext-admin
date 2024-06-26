<template>
  <div>
    <BasicTable @register="registerTable">
      <template #bodyCell="{ column, record }">
        <template v-if="column.key === 'action'">
          <TableAction
            :stop-button-propagation="true"
            :actions="[
              {
                auth: 'AbpOssManagement.OssObject.Download',
                ifShow: !record.isFolder,
                label: L('Objects:Download'),
                icon: 'ant-design:download-outlined',
                onClick: handleDownload.bind(null, record),
              },
              {
                label: L('Share'),
                icon: 'ant-design:share-alt-outlined',
                ifShow: shareEnabled,
                onClick: handleShare.bind(null, record),
              },
              {
                color: 'error',
                label: L('Delete'),
                icon: 'ant-design:delete-outlined',
                ifShow: deleteEnabled,
                onClick: handleDelete.bind(null, record),
              },
            ]"
          />
        </template>
      </template>
    </BasicTable>
    <FileShareModal @register="registerShareModal" />
  </div>
</template>

<script lang="ts" setup>
  import { computed, watchEffect } from 'vue';
  import { useLocalization } from '/@/hooks/abp/useLocalization';
  import { useMessage } from '/@/hooks/web/useMessage';
  import { BasicTable, TableAction, useTable } from '/@/components/Table';
  import { useModal } from '/@/components/Modal';
  import { getDataColumns } from './data';
  import { OssObject } from '/@/api/oss-management/objects/model';
  import { getList as getPrivates } from '/@/api/oss-management/files/private';
  import { getList as getPublices } from '/@/api/oss-management/files/public';
  import { generateOssUrl, deleteObject } from '/@/api/oss-management/objects';
  import FileShareModal from './FileShareModal.vue';

  const props = defineProps({
    selectGroup: {
      type: String,
      required: true,
      default: 'private',
    },
    selectPath: {
      type: String,
      required: true,
      default: '/',
    },
    deleteEnabled: {
      type: Boolean,
      required: true,
      default: false,
    },
  });
  const emit = defineEmits(['delete:file:private', 'delete:file:public', 'append:folder']);

  const { L } = useLocalization(['AbpOssManagement', 'AbpUi']);
  const { createConfirm, createMessage } = useMessage();
  const [registerTable, { setTableData, deleteTableDataRecord }] = useTable({
    rowKey: 'name',
    columns: getDataColumns(),
    title: L('FileList'),
    pagination: false,
    striped: false,
    useSearchForm: false,
    showTableSetting: true,
    tableSetting: {
      redo: false,
    },
    bordered: true,
    showIndexColumn: false,
    canResize: false,
    immediate: false,
    rowSelection: { type: 'checkbox' },
    actionColumn: {
      width: 240,
      title: L('Actions'),
      dataIndex: 'action',
    },
  });

  const shareEnabled = computed(() => {
    return props.selectGroup === 'private';
  });
  const bucket = computed(() => {
    switch (props.selectGroup) {
      case 'private':
        return 'users';
      case 'public':
        return 'public';
      default:
        return '';
    }
  });

  const [registerShareModal, { openModal }] = useModal();

  watchEffect(() => {
    props.selectGroup === 'private' && _fetchFileList(getPrivates);
    props.selectGroup === 'public' && _fetchFileList(getPublices);
  });

  function _fetchFileList(api: (...args: any) => Promise<ListResultDto<OssObject>>) {
    api({
      path: props.selectPath,
      maxResultCount: 100,
    }).then((res) => {
      const folders = res.items.filter((item) => item.isFolder);
      emit('append:folder', folders);
      setTableData(res.items.filter((item) => !item.isFolder));
    });
  }

  function handleDownload(record) {
    const link = document.createElement('a');
    link.style.display = 'none';
    link.href = generateOssUrl(bucket.value, record.path, record.name);
    link.setAttribute('download', record.name);
    document.body.appendChild(link);
    link.click();
  }

  function handleDelete(record) {
    createConfirm({
      iconType: 'warning',
      title: L('AreYouSure'),
      content: L('ItemWillBeDeletedMessage'),
      onOk: () => {
        return deleteObject({
          bucket: bucket.value,
          path: record.path,
          object: record.name,
        }).then(() => {
          createMessage.success(L('SuccessfullyDeleted'));
          deleteTableDataRecord(record.name);
          props.selectGroup === 'private' && emit('delete:file:private', record);
          props.selectGroup === 'public' && emit('delete:file:public', record);
        });
      },
    });
  }

  function handleShare(record) {
    openModal(true, record);
  }
</script>
