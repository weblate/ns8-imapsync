<!--
  Copyright (C) 2023 Nethesis S.r.l.
  SPDX-License-Identifier: GPL-3.0-or-later
-->
<template>
  <NsModal
    size="default"
    :visible="isShown"
    @modal-hidden="$emit('hide')"
    @primary-click="createTask"
  >
    <template slot="title"
      >{{
        !isEdit
          ? $t("tasks.create_task") + " "
          : $t("tasks.edit_task") + " "
      }}{{ task.localuser }}</template
    >
    <template slot="content">
      <cv-form>
        <template v-if="!isEdit">
          <NsComboBox
            v-model.trim="task.localuser"
            :autoFilter="true"
            :autoHighlight="true"
            :title="$t('tasks.local_user')"
            :label="$t('tasks.choose_local_user')"
            :options="enabled_mailboxes"
            :userInputLabel="$t('tasks.choose_local_user')"
            :acceptUserInput="false"
            :showItemType="true"
            :invalid-message="$t(error.enabled_mailboxes)"
            tooltipAlignment="start"
            tooltipDirection="top"
            ref="localuser"
          >
            <template slot="tooltip">
              {{ $t("tasks.choose_the_user_to_sync") }}
            </template>
          </NsComboBox>
        </template>
        <NsTextInput
          v-model.trim="task.remoteusername"
          :label="$t('tasks.remoteusername')"
          ref="remoteusername"
        />
        <NsTextInput
          v-model.trim="task.remotepassword"
          type="password"
          :label="$t('tasks.remotepassword')"
          ref="remotepassword"
        />
        <NsTextInput
          v-model.trim="task.remotehostname"
          :label="$t('tasks.remotehostname')"
          ref="remotehostname"
        />
        <NsTextInput
          v-model.trim="task.remoteport"
          type="number"
          :label="$t('tasks.remoteport')"
          ref="remoteport"
        />
        <cv-dropdown
          :light="true"
          :value="task.security"
          v-model="task.security"
          :up="false"
          :inline="false"
          :helper-text="$t('tasks.encryption_depends_remote_server')"
          :hide-selected="false"
          :label="$t('tasks.select_your_encryption')"
        >
          <cv-dropdown-item selected value="">{{
            $t("tasks.none")
          }}</cv-dropdown-item>
          <cv-dropdown-item value="tls">{{
            $t("tasks.use_starttls_default_143/tcp")
          }}</cv-dropdown-item>
          <cv-dropdown-item value="ssl">{{
            $t("tasks.use_imaps_default_993/tcp")
          }}</cv-dropdown-item>
        </cv-dropdown>
        <cv-text-area
          :label="$t('tasks.exclude_folder')"
          v-model.trim="task.exclude"
          ref="exclude"
          :placeholder="$t('tasks.write_one_exclusion_per_line')"
        >
        </cv-text-area>
        <NsToggle
          :label="$t('tasks.delete_on_local')"
          class="mg-left"
          value="delete"
          :form-item="true"
          v-model="task.delete"
          ref="delete"
        >
          <template slot="tooltip">
            <span v-html="$t('tasks.delete_tips')"></span>
          </template>
          <template slot="text-left">{{ $t("tasks.disabled") }}</template>
          <template slot="text-right">{{ $t("tasks.enabled") }}</template>
        </NsToggle>
        <NsToggle
          :label="$t('tasks.trashsync')"
          class="mg-left"
          value="trashsync"
          :form-item="true"
          v-model="task.trashsync"
          ref="trashsync"
        >
          <template slot="tooltip">
            <span v-html="$t('tasks.trashsync_tips')"></span>
          </template>
          <template slot="text-left">{{ $t("tasks.disabled") }}</template>
          <template slot="text-right">{{ $t("tasks.enabled") }}</template>
        </NsToggle>
      </cv-form>
    </template>
    <cv-row v-if="error.createTaks">
      <cv-column>
        <NsInlineNotification
          kind="error"
          :title="$t('action.createTask')"
          :description="error.createTask"
          :showCloseButton="false"
        />
      </cv-column>
    </cv-row>
    <template slot="secondary-button">{{ $t("common.cancel") }}</template>
    <template slot="primary-button">{{ $t("common.save") }}</template>
  </NsModal>
</template>

<script>
import to from "await-to-js";
import { UtilService, TaskService } from "@nethserver/ns8-ui-lib";
import { mapState } from "vuex";

export default {
  name: "CreateOrEditTask",
  mixins: [UtilService, TaskService],
  props: {
    isShown: Boolean,
    isEdit: Boolean,
    task: { type: [Object, null] },
    enabled_mailboxes: { type: Array },
  },

  data() {
    return {
      isValidated: false,
      previousValues: { Port: "", Security: "", hostname: "" },
      loading: {
        testImap: false,
        createTask: false,
        setCreateTask: false,
      },
      error: {
        enabled_mailboxe: "",
        testImap: "",
        createTask: "",
      },
    };
  },
  computed: {
    ...mapState(["instanceName", "core", "appName"]),
  },
  methods: {
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
            localuser: this.task.localuser,
            remotehostname: this.task.remotehostname,
            remotepassword: this.task.remotepassword,
            remoteport: Number(this.task.remoteport),
            remoteusername: this.task.remoteusername,
            security: this.task.security,
            delete: this.task.delete,
            exclude: this.task.exclude
              .split("\n")
              .map((item) => item.trim())
              .join(","),
            trashsync: this.task.trashsync,
          },
          extra: {
            title: this.$t("action." + taskAction),
            isNotificationHidden: true,
            eventId,
          },
        })
      );
      const err = res[0];
      // this.previousValues.Port = this.task.remoteport;
      // this.previousValues.Security = this.task.security;
      // this.previousValues.hostname = this.task.remotehostname;
      if (err) {
        console.error(`error creating task ${taskAction}`, err);
        this.error.setCreateTask = this.getErrorMessage(err);
        this.loading.setCreateTask = false;
        return;
      }
    },
    setCreateTaskAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.setCreateTask = this.$t("error.generic_error");
      this.loading.setCreateTask = false;
    },
    setCreateTaskCompleted() {
      this.loading.setCreateTask = false;
      this.task.localuser =="";
      this.$emit("hide");
      this.$emit("reloadtasks");
    },
  },
};
</script>

<style scoped lang="scss">
@import "../styles/carbon-utils";
</style>
