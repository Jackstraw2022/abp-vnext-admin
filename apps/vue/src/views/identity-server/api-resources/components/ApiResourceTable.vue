<template>
  <div>
    <BasicTable @register="registerTable">
      <template #toolbar>
        <Button type="primary" @click="handleAddNew">{{ L('AddNew') }}</Button>
      </template>
      <template #bodyCell="{ column, record }">
        <template v-if="column.key === 'enabled'">
          <CheckOutlined v-if="record.enabled" class="enable" />
          <CloseOutlined v-else class="disable" />
        </template>
        <template v-if="column.key === 'showInDiscoveryDocument'">
          <CheckOutlined v-if="record.showInDiscoveryDocument" class="enable" />
          <CloseOutlined v-else class="disable" />
        </template>
        <template v-else-if="column.key === 'action'">
          <TableAction
            :actions="[
              {
                auth: 'AbpIdentityServer.ApiResources.Update',
                icon: 'ant-design:edit-outlined',
                label: L('Resource:Edit'),
                onClick: handleEdit.bind(null, record),
              },
              {
                auth: 'AbpIdentityServer.ApiResources.Delete',
                color: 'error',
                icon: 'ant-design:delete-outlined',
                label: L('Resource:Delete'),
                onClick: handleDelete.bind(null, record),
              },
            ]"
          />
        </template>
      </template>
    </BasicTable>
    <ApiResourceModal @register="registerModal" @change="handleChange" />
  </div>
</template>

<script lang="ts" setup>
  import { Button } from 'ant-design-vue';
  import { CheckOutlined, CloseOutlined } from '@ant-design/icons-vue';
  import { useMessage } from '/@/hooks/web/useMessage';
  import { useLocalization } from '/@/hooks/abp/useLocalization';
  import { BasicTable, TableAction, useTable } from '/@/components/Table';
  import { useModal } from '/@/components/Modal';
  import { getDataColumns } from '../datas/TableData';
  import { getSearchFormSchemas } from '../datas/ModalData';
  import { deleteById, getList } from '/@/api/identity-server/api-resources';
  import { formatPagedRequest } from '/@/utils/http/abp/helper';
  import ApiResourceModal from './ApiResourceModal.vue';

  const { createMessage, createConfirm } = useMessage();
  const { L } = useLocalization('AbpIdentityServer');
  const [registerModal, { openModal, closeModal }] = useModal();
  const [registerTable, { reload }] = useTable({
    rowKey: 'id',
    title: L('DisplayName:ApiResources'),
    columns: getDataColumns(),
    api: getList,
    beforeFetch: formatPagedRequest,
    pagination: true,
    striped: false,
    useSearchForm: true,
    showTableSetting: true,
    bordered: true,
    showIndexColumn: false,
    canResize: true,
    immediate: true,
    canColDrag: true,
    formConfig: getSearchFormSchemas(),
    actionColumn: {
      width: 200,
      title: L('Actions'),
      dataIndex: 'action',
    },
  });

  function handleAddNew() {
    openModal(true, {});
  }

  function handleEdit(record) {
    openModal(true, record);
  }

  function handleChange() {
    closeModal();
    reload();
  }

  function handleDelete(record) {
    createConfirm({
      iconType: 'warning',
      title: L('AreYouSure'),
      content: L('ItemWillBeDeletedMessage'),
      okCancel: true,
      onOk: () => {
        return deleteById(record.id).then(() => {
          createMessage.success(L('SuccessfullyDeleted'));
          reload();
        });
      },
    });
  }
</script>
