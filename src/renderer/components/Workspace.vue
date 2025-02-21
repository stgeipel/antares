<template>
   <div v-show="isSelected" class="workspace column columns col-gapless">
      <WorkspaceExploreBar :connection="connection" :is-selected="isSelected" />
      <div v-if="workspace.connected" class="workspace-tabs column columns col-gapless">
         <ul
            id="tabWrap"
            ref="tabWrap"
            class="tab tab-block column col-12"
         >
            <li class="tab-item dropdown tools-dropdown">
               <a
                  class="tab-link workspace-tools-link dropdown-toggle"
                  tabindex="0"
                  :title="$t('word.tools')"
               >
                  <i class="mdi mdi-24px mdi-tools" />
               </a>
               <ul class="menu text-left text-uppercase">
                  <li v-if="workspace.customizations.processesList" class="menu-item">
                     <a class="c-hand p-vcentered" @click="showProcessesModal">
                        <i class="mdi mdi-memory mr-1 tool-icon" />
                        <span>{{ $t('message.processesList') }}</span>
                     </a>
                  </li>
                  <li
                     v-if="workspace.customizations.variables"
                     class="menu-item"
                     title="Coming..."
                  >
                     <a class="c-hand p-vcentered disabled">
                        <i class="mdi mdi-shape mr-1 tool-icon" />
                        <span>{{ $t('word.variables') }}</span>
                     </a>
                  </li>
                  <li
                     v-if="workspace.customizations.usersManagement"
                     class="menu-item"
                     title="Coming..."
                  >
                     <a class="c-hand p-vcentered disabled">
                        <i class="mdi mdi-account-group mr-1 tool-icon" />
                        <span>{{ $t('message.manageUsers') }}</span>
                     </a>
                  </li>
               </ul>
            </li>
            <li
               v-if="schemaChild && isSettingSupported"
               class="tab-item"
               :class="{'active': selectedTab === 'prop'}"
               @click="selectTab({uid: workspace.uid, tab: 'prop'})"
            >
               <a class="tab-link">
                  <i class="mdi mdi-18px mdi-tune-vertical-variant mr-1" />
                  <span :title="schemaChild">{{ $t('word.settings').toUpperCase() }}: {{ schemaChild }}</span>
               </a>
            </li>
            <li
               v-if="workspace.breadcrumbs.table || workspace.breadcrumbs.view"
               class="tab-item"
               :class="{'active': selectedTab === 'data'}"
               @click="selectTab({uid: workspace.uid, tab: 'data'})"
            >
               <a class="tab-link">
                  <i class="mdi mdi-18px mr-1" :class="workspace.breadcrumbs.table ? 'mdi-table' : 'mdi-table-eye'" />
                  <span :title="schemaChild">{{ $t('word.data').toUpperCase() }}: {{ schemaChild }}</span>
               </a>
            </li>
            <li
               v-for="tab of queryTabs"
               :key="tab.uid"
               class="tab-item"
               :class="{'active': selectedTab === tab.uid}"
               @click="selectTab({uid: workspace.uid, tab: tab.uid})"
               @mouseup.middle="closeTab(tab.uid)"
            >
               <a class="tab-link">
                  <i class="mdi mdi-18px mdi-code-tags mr-1" />
                  <span>
                     Query #{{ tab.index }}
                     <span
                        v-if="queryTabs.length > 1"
                        class="btn btn-clear"
                        :title="$t('word.close')"
                        @click.stop="closeTab(tab.uid)"
                     />
                  </span>
               </a>
            </li>
            <li class="tab-item">
               <a
                  class="tab-add"
                  :title="$t('message.openNewTab')"
                  @click="addTab"
               >
                  <i class="mdi mdi-24px mdi-plus" />
               </a>
            </li>
         </ul>
         <WorkspacePropsTab
            v-show="selectedTab === 'prop' && workspace.breadcrumbs.table"
            :is-selected="selectedTab === 'prop'"
            :connection="connection"
            :table="workspace.breadcrumbs.table"
         />
         <WorkspacePropsTabView
            v-show="selectedTab === 'prop' && workspace.breadcrumbs.view"
            :is-selected="selectedTab === 'prop'"
            :connection="connection"
            :view="workspace.breadcrumbs.view"
         />
         <WorkspacePropsTabTrigger
            v-show="selectedTab === 'prop' && workspace.breadcrumbs.trigger"
            :is-selected="selectedTab === 'prop'"
            :connection="connection"
            :trigger="workspace.breadcrumbs.trigger"
         />
         <WorkspacePropsTabRoutine
            v-show="selectedTab === 'prop' && workspace.breadcrumbs.procedure"
            :is-selected="selectedTab === 'prop'"
            :connection="connection"
            :routine="workspace.breadcrumbs.procedure"
         />
         <WorkspacePropsTabFunction
            v-show="selectedTab === 'prop' && workspace.breadcrumbs.function"
            :is-selected="selectedTab === 'prop'"
            :connection="connection"
            :function="workspace.breadcrumbs.function"
         />
         <WorkspacePropsTabScheduler
            v-show="selectedTab === 'prop' && workspace.breadcrumbs.scheduler"
            :is-selected="selectedTab === 'prop'"
            :connection="connection"
            :scheduler="workspace.breadcrumbs.scheduler"
         />
         <WorkspaceTableTab
            v-show="selectedTab === 'data'"
            :connection="connection"
            :table="workspace.breadcrumbs.table || workspace.breadcrumbs.view"
         />
         <WorkspaceQueryTab
            v-for="tab of queryTabs"
            :key="tab.uid"
            :tab="tab"
            :is-selected="selectedTab === tab.uid"
            :connection="connection"
         />
      </div>
      <ModalProcessesList
         v-if="isProcessesModal"
         :connection="connection"
         @close="hideProcessesModal"
      />
   </div>
</template>

<script>
import { mapGetters, mapActions } from 'vuex';
import Connection from '@/ipc-api/Connection';
import WorkspaceExploreBar from '@/components/WorkspaceExploreBar';
import WorkspaceQueryTab from '@/components/WorkspaceQueryTab';
import WorkspaceTableTab from '@/components/WorkspaceTableTab';
import WorkspacePropsTab from '@/components/WorkspacePropsTab';
import WorkspacePropsTabView from '@/components/WorkspacePropsTabView';
import WorkspacePropsTabTrigger from '@/components/WorkspacePropsTabTrigger';
import WorkspacePropsTabRoutine from '@/components/WorkspacePropsTabRoutine';
import WorkspacePropsTabFunction from '@/components/WorkspacePropsTabFunction';
import WorkspacePropsTabScheduler from '@/components/WorkspacePropsTabScheduler';
import ModalProcessesList from '@/components/ModalProcessesList';

export default {
   name: 'Workspace',
   components: {
      WorkspaceExploreBar,
      WorkspaceQueryTab,
      WorkspaceTableTab,
      WorkspacePropsTab,
      WorkspacePropsTabView,
      WorkspacePropsTabTrigger,
      WorkspacePropsTabRoutine,
      WorkspacePropsTabFunction,
      WorkspacePropsTabScheduler,
      ModalProcessesList
   },
   props: {
      connection: Object
   },
   data () {
      return {
         hasWheelEvent: false,
         isProcessesModal: false
      };
   },
   computed: {
      ...mapGetters({
         selectedWorkspace: 'workspaces/getSelected',
         getWorkspace: 'workspaces/getWorkspace'
      }),
      workspace () {
         return this.getWorkspace(this.connection.uid);
      },
      isSelected () {
         return this.selectedWorkspace === this.connection.uid;
      },
      isSettingSupported () {
         if (this.workspace.breadcrumbs.table && this.workspace.customizations.tableSettings) return true;
         if (this.workspace.breadcrumbs.view && this.workspace.customizations.viewSettings) return true;
         if (this.workspace.breadcrumbs.trigger && this.workspace.customizations.triggerSettings) return true;
         if (this.workspace.breadcrumbs.procedure && this.workspace.customizations.routineSettings) return true;
         if (this.workspace.breadcrumbs.function && this.workspace.customizations.functionSettings) return true;
         if (this.workspace.breadcrumbs.scheduler && this.workspace.customizations.schedulerSettings) return true;
         return false;
      },
      selectedTab () {
         if (
            (
               this.workspace.breadcrumbs.table === null &&
               this.workspace.breadcrumbs.view === null &&
               this.workspace.breadcrumbs.trigger === null &&
               this.workspace.breadcrumbs.procedure === null &&
               this.workspace.breadcrumbs.function === null &&
               this.workspace.breadcrumbs.scheduler === null &&
               ['data', 'prop'].includes(this.workspace.selected_tab)
            ) ||
            (
               this.workspace.breadcrumbs.table === null &&
               this.workspace.breadcrumbs.view === null &&
               this.workspace.selected_tab === 'data'
            )
         )
            return this.queryTabs[0].uid;

         return this.queryTabs.find(tab => tab.uid === this.workspace.selected_tab) ||
         ['data', 'prop'].includes(this.workspace.selected_tab)
            ? this.workspace.selected_tab
            : this.queryTabs[0].uid;
      },
      queryTabs () {
         return this.workspace.tabs.filter(tab => tab.type === 'query');
      },
      schemaChild () {
         for (const key in this.workspace.breadcrumbs) {
            if (key === 'schema') continue;
            if (this.workspace.breadcrumbs[key]) return this.workspace.breadcrumbs[key];
         }
         return false;
      }
   },
   async created () {
      await this.addWorkspace(this.connection.uid);
      const isInitiated = await Connection.checkConnection(this.connection.uid);
      if (isInitiated)
         this.connectWorkspace(this.connection);
   },
   mounted () {
      if (this.$refs.tabWrap) {
         this.$refs.tabWrap.addEventListener('wheel', e => {
            if (e.deltaY > 0) this.$refs.tabWrap.scrollLeft += 50;
            else this.$refs.tabWrap.scrollLeft -= 50;
         });
      }
   },
   methods: {
      ...mapActions({
         addWorkspace: 'workspaces/addWorkspace',
         connectWorkspace: 'workspaces/connectWorkspace',
         removeConnected: 'workspaces/removeConnected',
         selectTab: 'workspaces/selectTab',
         newTab: 'workspaces/newTab',
         removeTab: 'workspaces/removeTab'
      }),
      addTab () {
         this.newTab({ uid: this.connection.uid });

         if (!this.hasWheelEvent) {
            this.$refs.tabWrap.addEventListener('wheel', e => {
               if (e.deltaY > 0) this.$refs.tabWrap.scrollLeft += 50;
               else this.$refs.tabWrap.scrollLeft -= 50;
            });
            this.hasWheelEvent = true;
         }
      },
      closeTab (tUid) {
         if (this.queryTabs.length === 1) return;
         this.removeTab({ uid: this.connection.uid, tab: tUid });
      },
      showProcessesModal () {
         this.isProcessesModal = true;
      },
      hideProcessesModal () {
         this.isProcessesModal = false;
      }
   }
};
</script>

<style lang="scss">
.workspace {
  padding: 0;
  margin: 0;

  .workspace-tabs {
    overflow: hidden;
    height: calc(100vh - #{$excluding-size});

    .tab-block {
      background: $bg-color-light;
      margin-top: 0;
      flex-direction: row;
      align-items: flex-start;
      flex-wrap: nowrap;
      overflow: auto;
      margin-bottom: 0;

      &::-webkit-scrollbar {
        width: 2px;
        height: 2px;
      }

      .tab-item {
        max-width: 12rem;
        width: fit-content;
        flex: initial;

        > a {
          padding: 0.2rem 0.8rem;
          color: $body-font-color;
          cursor: pointer;
          display: flex;
          align-items: center;
          opacity: 0.7;
          transition: opacity 0.2s;

          &:hover {
            opacity: 1;
          }

          &.tab-add {
            padding: 0.2rem 0.4rem;
            margin-top: 2px;
            border: 0;
          }

          > span {
            overflow: hidden;
            white-space: nowrap;
            text-overflow: ellipsis;
            padding: 0 0.2rem;
          }
        }

        &.active a {
          opacity: 1;
        }

        &.tools-dropdown {
          .tab-link:focus {
            color: $primary-color;
            opacity: 1;
            outline: 0;
            box-shadow: none;
          }

          .menu {
            min-width: 100%;

            .menu-item a {
              border-radius: 0.1rem;
              color: inherit;
              display: block;
              margin: 0 -0.4rem;
              padding: 0.2rem 0.4rem;
              text-decoration: none;
              white-space: nowrap;
              border: 0;

              &:hover {
                color: $primary-color;
                background: $bg-color-gray;
              }

              .tool-icon {
                line-height: 1;
                display: inline-block;
                font-size: 20px;
              }
            }
          }

          z-index: 9;
          position: absolute;
        }

        &.tools-dropdown + .tab-item {
          margin-left: 56px;
        }

        .workspace-tools-link {
          padding-bottom: 0;
          padding-top: 0.3rem;
        }
      }
    }
  }

  .workspace-query-results {
    overflow: auto;
    white-space: nowrap;

    .table {
      width: auto;
      border-collapse: separate;

      .th {
        position: sticky;
        top: 0;
        background: $bg-color;
        border: 1px solid;
        border-left: none;
        border-bottom-width: 2px;
        border-color: $bg-color-light;
        padding: 0;
        font-weight: 700;
        font-size: 0.7rem;
        z-index: 1;

        > div {
          padding: 0.1rem 0.4rem;
          min-width: -webkit-fill-available;
        }
      }

      .td {
        border-right: 1px solid;
        border-bottom: 1px solid;
        border-color: $bg-color-light;
        padding: 0 0.4rem;
        text-overflow: ellipsis;
        max-width: 200px;
        white-space: nowrap;
        overflow: hidden;
        font-size: 0.7rem;
        position: relative;

        &:focus {
          box-shadow: inset 0 0 0 1px $body-font-color;
          background: rgba($color: #000, $alpha: 0.3);
          outline: none;
        }
      }
    }
  }
}
</style>
