<template>
   <BaseContextMenu
      :context-event="contextEvent"
      @close-context="closeContext"
   >
      <div
         v-if="selectedRows.length === 1"
         class="context-element"
         @click="setNull"
      >
         <span class="d-flex">
            <i class="mdi mdi-18px mdi-null text-light pr-1" /> {{ $t('message.setNull') }}
         </span>
      </div>
      <div class="context-element" @click="showConfirmModal">
         <span class="d-flex">
            <i class="mdi mdi-18px mdi-delete text-light pr-1" /> {{ $tc('message.deleteRows', selectedRows.length) }}
         </span>
      </div>

      <ConfirmModal
         v-if="isConfirmModal"
         @confirm="deleteRows"
         @hide="hideConfirmModal"
      >
         <template :slot="'header'">
            <div class="d-flex">
               <i class="mdi mdi-24px mdi-delete mr-1" /> {{ $tc('message.deleteRows', selectedRows.length) }}
            </div>
         </template>
         <div :slot="'body'">
            <div class="mb-2">
               {{ $tc('message.confirmToDeleteRows', selectedRows.length) }}
            </div>
         </div>
      </ConfirmModal>
   </BaseContextMenu>
</template>

<script>
import BaseContextMenu from '@/components/BaseContextMenu';
import ConfirmModal from '@/components/BaseConfirmModal';

export default {
   name: 'WorkspaceQueryTableContext',
   components: {
      BaseContextMenu,
      ConfirmModal
   },
   props: {
      contextEvent: MouseEvent,
      selectedRows: Array
   },
   data () {
      return {
         isConfirmModal: false
      };
   },
   computed: {
   },
   methods: {
      showConfirmModal () {
         this.isConfirmModal = true;
      },
      hideConfirmModal () {
         this.isConfirmModal = false;
      },
      closeContext () {
         this.$emit('close-context');
      },
      deleteRows () {
         this.$emit('delete-selected');
         this.closeContext();
      },
      setNull () {
         this.$emit('set-null');
         this.closeContext();
      }
   }
};
</script>
