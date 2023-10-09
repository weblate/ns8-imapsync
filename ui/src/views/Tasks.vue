<!--
  Copyright (C) 2023 Nethesis S.r.l.
  SPDX-License-Identifier: GPL-3.0-or-later
-->
<template>
  <div>
    <cv-grid fullWidth>
      <cv-row>
        <cv-column class="page-title">
          <h2>
            {{ $t("tasks.title") }}
            <cv-interactive-tooltip
              alignment="start"
              direction="right"
              class="tooltip info mg-left-sm"
            >
              <template slot="trigger"></template>
              <template slot="content">
                <div v-html="$t('tasks.title_tooltip')"></div>
              </template>
            </cv-interactive-tooltip>
          </h2>
        </cv-column>
      </cv-row>
      <cv-row>
        <cv-column>
          <NsInlineNotification
            kind="warning"
            :title="core.$t('common.use_landscape_mode')"
            :description="core.$t('common.use_landscape_mode_description')"
            class="landscape-warning"
          />
        </cv-column>
      </cv-row>
      <cv-row class="toolbar">
        <cv-column>
          <template>
            <NsButton
              kind="secondary"
              :icon="Restart20"
              @click="listTasks"
              :disabled="loading.listTasks || loading.setDeleteTask"
              >{{ $t("tasks.reload_tasks") }}
            </NsButton>
          </template>
          <template v-if="tasks.length">
            <NsButton
              kind="secondary"
              class="mg-left"
              :icon="Play20"
              @click="startAllTasks"
              :disabled="loading.listTasks || loading.setDeleteTask"
              >{{ $t("tasks.start_all") }}
            </NsButton>
            <NsButton
              kind="secondary"
              class="mg-left"
              :icon="Stop20"
              @click="stopAllTasks"
              :disabled="loading.listTasks || loading.setDeleteTask"
              >{{ $t("tasks.stop_all") }}
            </NsButton>
          </template>
        </cv-column>
      </cv-row>
      <cv-row>
        <cv-column>
          <cv-tile light>
            <NsDataTable
              :allRows="tasks"
              :columns="i18nTableColumns"
              :rawColumns="tableColumns"
              :sortable="true"
              :pageSizes="[10, 25, 50, 100]"
              :overflow-menu="true"
              :isSearchable="check_tasks"
              :searchPlaceholder="$t('tasks.search_tasks')"
              :searchClearLabel="core.$t('common.clear_search')"
              :noSearchResultsLabel="core.$t('common.no_search_results')"
              :noSearchResultsDescription="
                core.$t('common.no_search_results_description')
              "
              :isLoading="loading.listTasks || loading.setDeleteTask"
              :skeletonRows="5"
              :isErrorShown="!!error.listTasks"
              :errorTitle="$t('action.Tasks_list_error')"
              :errorDescription="error.listTasks"
              :itemsPerPageLabel="core.$t('pagination.items_per_page')"
              :rangeOfTotalItemsLabel="
                core.$t('pagination.range_of_total_items')
              "
              :ofTotalPagesLabel="core.$t('pagination.of_total_pages')"
              :backwardText="core.$t('pagination.previous_page')"
              :forwardText="core.$t('pagination.next_page')"
              :pageNumberLabel="core.$t('pagination.page_number')"
              @updatePage="tablePage = $event"
            >
              <template slot="empty-state">
                <NsEmptyState :title="$t('tasks.no_tasks')">
                </NsEmptyState>
              </template>
              <template slot="data">
                <cv-data-table-row
                  v-for="(row, rowIndex) in tasks"
                  :key="`${rowIndex}`"
                  :value="`${rowIndex}`"
                >
                  <cv-data-table-row>
                    <div class="mg-top mg-left gray">
                      {{
                        row.mailbox
                      }}
                    </div>
                  </cv-data-table-row>
                  <cv-data-table-cell>
                    {{ row.remoteusername }}
                  </cv-data-table-cell>
                  <cv-data-table-cell>
                    {{ row.remotehostname }}
                  </cv-data-table-cell>
                  <cv-data-table-cell>
                    {{ row.service ? $t('tasks.running') : row.remoteusername === "" ? $t('tasks.not_configured') : $t('tasks.stopped') }}
                  </cv-data-table-cell>
                  <cv-data-table-cell class="table-overflow-menu-cell">
                    <cv-overflow-menu
                      flip-menu
                      class="table-overflow-menu"
                      :data-test-id="row.mailbox + '-menu'"
                    >
                      <cv-overflow-menu-item
                        @click="toggleCreateOrEditTask(row)"
                        :data-test-id="row.mailbox + '-edit-task'"
                      >
                        <NsMenuItem
                          :icon="row.remoteusername === '' ? Add20 : Edit20"
                          :label="row.remoteusername === '' ? $t('tasks.create') : $t('tasks.edit')"
                        />
                      </cv-overflow-menu-item>
                      <cv-overflow-menu-item v-if="row.remoteusername !== ''"
                        @click="toggleActionTask(row)"
                        :data-test-id="row.mailbox + '-action-task'"
                      >
                        <NsMenuItem
                          :icon="row.service ? Stop20 : Play20"
                          :label="row.service ? $t('tasks.stop'): $t('tasks.start')"
                        />
                      </cv-overflow-menu-item>
                      <cv-overflow-menu-item v-if="row.remoteusername !== ''"
                        @click="toggleDeleteTask(row)"
                        :data-test-id="row.mailbox + '-delete-task'"
                      >
                        <NsMenuItem
                          :icon="TrashCan20"
                          :label="$t('tasks.delete')"
                        />
                      </cv-overflow-menu-item>
                    </cv-overflow-menu>
                  </cv-data-table-cell>
                </cv-data-table-row>
              </template>
            </NsDataTable>
          </cv-tile>
        </cv-column>
      </cv-row>
    </cv-grid>
    <ConfirmDeleteTask
      :isShown="isShownConfirmDeleteTask"
      :task="currentTask"
      :core="core"
      @hide="hideConfirmDeleteTask"
      @confirm="setDeleteTask(false)"
    />
    <CreateOrEditTask
      :isShown="isShownCreateOrEditTask"
      :task="currentTask"
      :core="core"
      @hide="hideCreateOrEditTask"
      @confirm="createTask(false)"
    />
  </div>
</template>

<script>
import { mapState } from "vuex";
import {
  QueryParamService,
  UtilService,
  IconService,
  TaskService,
  DateTimeService,
} from "@nethserver/ns8-ui-lib";
import to from "await-to-js";
import ConfirmDeleteTask from "@/components/ConfirmDeleteTask";
import CreateOrEditTask from "@/components/CreateOrEditTask";
import Play20 from "@carbon/icons-vue/es/play--outline/20";
import Stop20 from "@carbon/icons-vue/es/stop--outline/20";
import Add20 from "@carbon/icons-vue/es/task--add/20";
export default {
  name: "Tasks",
  components: {
    ConfirmDeleteTask,
    CreateOrEditTask,
  },
  mixins: [
    QueryParamService,
    UtilService,
    IconService,
    TaskService,
    DateTimeService,
  ],
  pageTitle() {
    return this.$t("tasks.title") + " - " + this.appName;
  },
  data() {
    return {
      q: {
        page: "tasks",
      },
      Play20,
      Stop20,
      Add20,
      urlCheckInterval: null,
      tablePage: [],
      tableColumns: ["mailbox", "remoteusername", "remotehostname","task_status"],
      tasks: [],
      check_tasks: false,
      isShownConfirmDeleteTask: false,
      isShownCreateOrEditTask: false,
      previousValues:{
        Port: "",
        Security: "",
        hostname: ""
      },
      currentTask: {
        mailbox: "",
        remoteusername: "",
        remotehostname: "",
        remotepassword:"",
        service: false,
        remoteport: "",
        security:"",
        delete: false,
        exclude:"",
        trashsync:false
      },
      loading: {
        listTasks: false,
        setDeleteTask: false
      },
      error: {
        listTasks: "",
        setDeleteTask: "",
        startAllTasks: "",
        stopAllTasks: "",
      },
    };
  },
  computed: {
    ...mapState(["instanceName", "core", "appName"]),
    i18nTableColumns() {
      return this.tableColumns.map((column) => {
        return this.$t("tasks.col_" + column);
      });
    },
  },
  beforeRouteEnter(to, from, next) {
    next((vm) => {
      vm.watchQueryData(vm);
      vm.urlCheckInterval = vm.initUrlBindingForApp(vm, vm.q.page);
    });
  },
  beforeRouteLeave(to, from, next) {
    clearInterval(this.urlCheckInterval);
    next();
  },
  created() {
    this.listTasks();
  },
  methods: {
    async listTasks() {
      // we push after object to tasks
      // we have to set to zero at first
      this.tasks = [];
      const taskAction = "list-tasks";
      const eventId = this.getUuid();
      this.loading.listTasks = true;
      // register to task events
      this.core.$root.$once(
        `${taskAction}-aborted-${eventId}`,
        this.listTasksAborted
      );
      this.core.$root.$once(
        `${taskAction}-completed-${eventId}`,
        this.listTasksCompleted
      );
      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          extra: {
            title: this.$t("action." + taskAction),
            isNotificationHidden: true,
            eventId,
          },
        })
      );
      const err = res[0];

      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        const errMessage = this.getErrorMessage(err);
        this.error.listTasks = errMessage;
        this.loading.listTasks = false;
      }
    },
    listTasksAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.listTasks = this.$t("error.generic_error");
      this.loading.listTasks = false;
    },
    listTasksCompleted(taskContext, taskResult) {
      let Config = taskResult.output;
      Config.user_properties.forEach((task) => {
          this.tasks.push({
            mailbox: task.mailbox,
            service: task.service_running,
            remotehostname: task.props.remotehostname,
            remotepassword: task.props.remotepassword,
            remoteport: task.props.remoteport,
            remoteusername: task.props.remoteusername,
            security: task.props.security,
            delete: task.props.delete,
            exclude: task.props.exclude.split('|').join('\n'),
            trashsync: task.props.trashsync
          });
      });
      this.check_tasks = this.tasks.length ? true : false;
      this.loading.listTasks = false;
    },
    toggleCreateOrEditTask(task) {
      this.currentTask = task;
      if (this.previousValues.Port) {
        this.currentTask.remoteport = this.previousValues.Port;
        this.currentTask.security = this.previousValues.Security;
        this.currentTask.remotehostname = this.previousValues.hostname;
      }
      this.showCreateOrEditTask();
    },
    showCreateOrEditTask() {
      this.isShownCreateOrEditTask = true;
    },
    hideCreateOrEditTask() {
      this.isShownCreateOrEditTask = false;
    },
    toggleDeleteTask(task) {
      this.currentTask = task;
      this.showConfirmDeleteTask();
    },
    showConfirmDeleteTask() {
      this.isShownConfirmDeleteTask = true;
    },
    hideConfirmDeleteTask() {
      this.isShownConfirmDeleteTask = false;
    },
    async setDeleteTask() {
      this.loading.setDeleteTask = true;
      this.error.setDeleteTask = "";
      const taskAction = "delete-task";
      const eventId = this.getUuid();
      // register to task error
      this.core.$root.$once(
        `${taskAction}-aborted-${eventId}`,
        this.setDeleteTaskAborted
      );
      // register to task completion
      this.core.$root.$once(
        `${taskAction}-completed-${eventId}`,
        this.setDeleteTaskCompleted
      );
      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          data: {
            localuser: this.currentTask.mailbox,
          },
          extra: {
            title: this.$t("action." + taskAction),
            isNotificationHidden: true,
            eventId,
          },
        })
      );
      const err = res[0];

      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.setDeleteTask = this.getErrorMessage(err);
        this.loading.setDeleteTask = false;
        return;
      }
      this.hideConfirmDeleteTask();
    },
    setDeleteTaskAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.setDeleteTask = this.$t("error.generic_error");
      this.loading.setDeleteTask = false;
    },
    setDeleteTaskCompleted() {
      this.loading.setDeleteTask = false;
      this.hideConfirmDeleteTask();
      this.listTasks();
    },
    async toggleActionTask(task) {
      this.loading.toggleActionTask = true;
      this.error.toggleActionTask = "";
      const taskAction = task.service ? "stop-task" : "start-task";
      const eventId = this.getUuid();
      // register to task error
      this.core.$root.$once(
        `${taskAction}-aborted-${eventId}`,
        this.toggleActionTaskAborted
      );
      // register to task completion
      this.core.$root.$once(
        `${taskAction}-completed-${eventId}`,
        this.toggleActionTaskCompleted
      );
      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          data: {
            localuser: task.mailbox,
          },
          extra: {
            title: this.$t("action." + taskAction),
            isNotificationHidden: true,
            eventId,
          },
        })
      );
      const err = res[0];

      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.toggleActionTask = this.getErrorMessage(err);
        this.loading.toggleActionTask = false;
        return;
      }
    },
    toggleActionTaskAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.toggleActionTask = this.$t("error.generic_error");
      this.loading.toggleActionTask = false;
    },
    toggleActionTaskCompleted() {
      this.loading.toggleActionTask = false;
      this.listTasks();
    },
    async createTask() {
      this.loading.createTask = true;
      this.error.createTask = "";
      const taskAction = "create-task";
      const eventId = this.getUuid();
      // register to task error
      this.core.$root.$once(
        `${taskAction}-aborted-${eventId}`,
        this.setCreateTaskAborted
      );
      // register to task completion
      this.core.$root.$once(
        `${taskAction}-completed-${eventId}`,
        this.setCreateTaskCompleted
      );
      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          data: {
            localuser: this.currentTask.mailbox,
            remotehostname: this.currentTask.remotehostname,
            remotepassword: this.currentTask.remotepassword,
            remoteport: this.currentTask.remoteport,
            remoteusername: this.currentTask.remoteusername,
            security: this.currentTask.security,
            delete: this.currentTask.delete,
            exclude: this.currentTask.exclude.split('\n').map(item => item.trim()).join(','),
            trashsync: this.currentTask.trashsync
          },
          extra: {
            title: this.$t("action." + taskAction),
            isNotificationHidden: true,
            eventId,
          },
        })
      );
      const err = res[0];
      this.previousValues.Port = this.currentTask.remoteport;
      this.previousValues.Security = this.currentTask.security;
      this.previousValues.hostname = this.currentTask.remotehostname;
      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.setCreateTask = this.getErrorMessage(err);
        this.loading.setCreateTask = false;
        return;
      }
      // this.hideCreateOrEditTask();
    },
    setCreateTaskAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.setCreateTask = this.$t("error.generic_error");
      this.loading.setCreateTask = false;
    },
    setCreateTaskCompleted() {
      this.loading.setCreateTask = false;
      this.hideCreateOrEditTask();
      this.listTasks();
    },
    async startAllTasks() {
      this.loading.startAllTasks = true;
      this.error.startAllTasks = "";
      const taskAction = "start-all-tasks";
      const eventId = this.getUuid();
      // register to task error
      this.core.$root.$once(
        `${taskAction}-aborted-${eventId}`,
        this.setStartAllTasksAborted
      );
      // register to task completion
      this.core.$root.$once(
        `${taskAction}-completed-${eventId}`,
        this.setStartAllTasksCompleted
      );
      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          data: {
          },
          extra: {
            title: this.$t("action." + taskAction),
            isNotificationHidden: true,
            eventId,
          },
        })
      );
      const err = res[0];

      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.startAllTasks = this.getErrorMessage(err);
        this.loading.startAllTasks = false;
        return;
      }
    },
    setStartAllTasksAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.startAllTasks = this.$t("error.generic_error");
      this.loading.startAllTasks = false;
    },
    setStartAllTasksCompleted() {
      this.loading.startAllTasks = false;
      this.listTasks();
    },
    async stopAllTasks() {
      this.loading.stopAllTasks = true;
      this.error.stopAllTasks = "";
      const taskAction = "stop-all-tasks";
      const eventId = this.getUuid();
      // register to task error
      this.core.$root.$once(
        `${taskAction}-aborted-${eventId}`,
        this.setStopAllTasksAborted
      );
      // register to task completion
      this.core.$root.$once(
        `${taskAction}-completed-${eventId}`,
        this.setStopAllTasksCompleted
      );
      const res = await to(
        this.createModuleTaskForApp(this.instanceName, {
          action: taskAction,
          data: {
          },
          extra: {
            title: this.$t("action." + taskAction),
            isNotificationHidden: true,
            eventId,
          },
        })
      );
      const err = res[0];

      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.stopAllTasks = this.getErrorMessage(err);
        this.loading.stopAllTasks = false;
        return;
      }
    },
    setStopAllTasksAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.stopAllTasks = this.$t("error.generic_error");
      this.loading.stopAllTasks = false;
    },
    setStopAllTasksCompleted() {
      this.loading.stopAllTasks = false;
      this.listTasks();
    },
  },
};
</script>

<style scoped lang="scss">
@import "../styles/carbon-utils";

.quota-stats {
  font-size: 90%;
  display: flex;
  justify-content: space-between;
}
.mg-top {
  margin-top: 1em;
}
.mg-left {
  margin-left: 1em;
}
.gray {
  color: #525252;
}
</style>
