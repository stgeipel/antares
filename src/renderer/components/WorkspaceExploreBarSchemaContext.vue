<template>
   <BaseContextMenu
      :context-event="contextEvent"
      @close-context="closeContext"
   >
      <div class="context-element">
         <span class="d-flex"><i class="mdi mdi-18px mdi-plus text-light pr-1" /> {{ $t('word.add') }}</span>
         <i class="mdi mdi-18px mdi-chevron-right text-light pl-1" />
         <div class="context-submenu">
            <div
               v-if="workspace.customizations.tableAdd"
               class="context-element"
               @click="showCreateTableModal"
            >
               <span class="d-flex"><i class="mdi mdi-18px mdi-table text-light pr-1" /> {{ $t('word.table') }}</span>
            </div>
            <div
               v-if="workspace.customizations.viewAdd"
               class="context-element"
               @click="showCreateViewModal"
            >
               <span class="d-flex"><i class="mdi mdi-18px mdi-table-eye text-light pr-1" /> {{ $t('word.view') }}</span>
            </div>
            <div
               v-if="workspace.customizations.triggerAdd"
               class="context-element"
               @click="showCreateTriggerModal"
            >
               <span class="d-flex"><i class="mdi mdi-18px mdi-table-cog text-light pr-1" /> {{ $tc('word.trigger', 1) }}</span>
            </div>
            <div
               v-if="workspace.customizations.routineAdd"
               class="context-element"
               @click="showCreateRoutineModal"
            >
               <span class="d-flex"><i class="mdi mdi-18px mdi-sync-circle pr-1" /> {{ $tc('word.storedRoutine', 1) }}</span>
            </div>
            <div
               v-if="workspace.customizations.functionAdd"
               class="context-element"
               @click="showCreateFunctionModal"
            >
               <span class="d-flex"><i class="mdi mdi-18px mdi-arrow-right-bold-box pr-1" /> {{ $tc('word.function', 1) }}</span>
            </div>
            <div
               v-if="workspace.customizations.schedulerAdd"
               class="context-element"
               @click="showCreateSchedulerModal"
            >
               <span class="d-flex"><i class="mdi mdi-18px mdi-calendar-clock text-light pr-1" /> {{ $tc('word.scheduler', 1) }}</span>
            </div>
         </div>
      </div>
      <div
         v-if="workspace.customizations.schemaEdit"
         class="context-element"
         @click="showEditModal"
      >
         <span class="d-flex"><i class="mdi mdi-18px mdi-database-edit text-light pr-1" /> {{ $t('word.edit') }}</span>
      </div>
      <div class="context-element" @click="showDeleteModal">
         <span class="d-flex"><i class="mdi mdi-18px mdi-database-remove text-light pr-1" /> {{ $t('word.delete') }}</span>
      </div>

      <ConfirmModal
         v-if="isDeleteModal"
         @confirm="deleteSchema"
         @hide="hideDeleteModal"
      >
         <template slot="header">
            <div class="d-flex">
               <i class="mdi mdi-24px mdi-database-remove mr-1" /> {{ $t('message.deleteSchema') }}
            </div>
         </template>
         <div slot="body">
            <div class="mb-2">
               {{ $t('message.deleteCorfirm') }} "<b>{{ selectedDatabase }}</b>"?
            </div>
         </div>
      </ConfirmModal>
      <ModalEditSchema
         v-if="isEditModal"
         :selected-database="selectedDatabase"
         @close="hideEditModal"
      />
   </BaseContextMenu>
</template>

<script>
import { mapGetters, mapActions } from 'vuex';
import BaseContextMenu from '@/components/BaseContextMenu';
import ConfirmModal from '@/components/BaseConfirmModal';
import ModalEditSchema from '@/components/ModalEditSchema';
import Schema from '@/ipc-api/Schema';

export default {
   name: 'WorkspaceExploreBarSchemaContext',
   components: {
      BaseContextMenu,
      ConfirmModal,
      ModalEditSchema
   },
   props: {
      contextEvent: MouseEvent,
      selectedDatabase: String
   },
   data () {
      return {
         isDeleteModal: false,
         isEditModal: false
      };
   },
   computed: {
      ...mapGetters({
         selectedWorkspace: 'workspaces/getSelected',
         getWorkspace: 'workspaces/getWorkspace'
      }),
      workspace () {
         return this.getWorkspace(this.selectedWorkspace);
      }
   },
   methods: {
      ...mapActions({
         addNotification: 'notifications/addNotification',
         changeBreadcrumbs: 'workspaces/changeBreadcrumbs'
      }),
      showCreateTableModal () {
         this.$emit('show-create-table-modal');
      },
      showCreateViewModal () {
         this.$emit('show-create-view-modal');
      },
      showCreateTriggerModal () {
         this.$emit('show-create-trigger-modal');
      },
      showCreateRoutineModal () {
         this.$emit('show-create-routine-modal');
      },
      showCreateFunctionModal () {
         this.$emit('show-create-function-modal');
      },
      showCreateSchedulerModal () {
         this.$emit('show-create-scheduler-modal');
      },
      showDeleteModal () {
         this.isDeleteModal = true;
      },
      hideDeleteModal () {
         this.isDeleteModal = false;
      },
      showEditModal () {
         this.isEditModal = true;
      },
      hideEditModal () {
         this.isEditModal = false;
         this.closeContext();
      },
      closeContext () {
         this.$emit('close-context');
      },
      async deleteSchema () {
         try {
            const { status, response } = await Schema.deleteSchema({
               uid: this.selectedWorkspace,
               database: this.selectedDatabase
            });

            if (status === 'success') {
               if (this.selectedDatabase === this.workspace.breadcrumbs.schema)
                  this.changeBreadcrumbs({ schema: null });

               this.closeContext();
               this.$emit('reload');
            }
            else
               this.addNotification({ status: 'error', message: response });
         }
         catch (err) {
            this.addNotification({ status: 'error', message: err.stack });
         }
      }
   }
};
</script>
<style lang="scss" scoped>
.context-submenu {
  min-width: 150px !important;
}
</style>
