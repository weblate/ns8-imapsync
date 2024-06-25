<!--
  Copyright (C) 2023 Nethesis S.r.l.
  SPDX-License-Identifier: GPL-3.0-or-later
-->
<template>
  <NsModal
    size="default"
    :visible="isShown"
    @modal-hidden="onModalHidden"
    @primary-click="createTask"
    :primary-button-disabled="loading.createTask"
  >
    <template slot="title">{{
      !isEdit
        ? $t("tasks.create_task") + " "
        : $t("tasks.edit_task") + " " + task.localuser
    }}</template>
    <template slot="content">
      <cv-form>
        <cv-row v-if="error.createTask">
          <cv-column>
            <NsInlineNotification
              kind="error"
              :title="$t('tasks.create_task')"
              :description="error.createTask"
              :showCloseButton="false"
            />
          </cv-column>
        </cv-row>
        <NsComboBox
          v-if="!isEdit"
          :options="enabled_mailboxes"
          v-model.trim="task.localuser"
          :autoFilter="true"
          :autoHighlight="true"
          :title="$t('tasks.local_user')"
          :label="$t('tasks.choose_local_user')"
          :userInputLabel="$t('tasks.choose_local_user')"
          :acceptUserInput="true"
          :showItemType="true"
          :invalid-message="$t(error.localuser)"
          tooltipAlignment="start"
          tooltipDirection="top"
          ref="localuser"
        >
          <template slot="tooltip">
            {{ $t("tasks.choose_the_user_to_sync") }}
          </template>
        </NsComboBox>
        <NsTextInput
          v-model.trim="task.remoteusername"
          :label="$t('tasks.remoteusername')"
          ref="remoteusername"
          :invalid-message="$t(error.remoteusername)"
        />
        <NsTextInput
          v-model.trim="task.remotepassword"
          type="password"
          :label="$t('tasks.remotepassword')"
          ref="remotepassword"
          :invalid-message="$t(error.remotepassword)"
        />
        <NsTextInput
          v-model.trim="task.remotehostname"
          :label="$t('tasks.remotehostname')"
          placeholder="imap.domain.com"
          ref="remotehostname"
          :invalid-message="$t(error.remotehostname)"
        />
        <NsTextInput
          v-model.trim="task.remoteport"
          type="number"
          :label="$t('tasks.remoteport')"
          ref="remoteport"
          :invalid-message="$t(error.remoteport)"
        />
        <cv-dropdown
          :light="true"
          class="max-dropdown-width"
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
        <span class="mg-bottom">
          {{ $t("tasks.synchronize_folders") }}
          <cv-tooltip
            alignment="start"
            direction="bottom"
            :tip="$t('tasks.synchronize_folders_explanation')"
            class="info mg-bottom"
          >
          </cv-tooltip>
        </span>
        <cv-radio-group vertical class="mg-bottom mg-left">
          <cv-radio-button
            :name="'radio-group-foldersynchronization'"
            :label="$t('tasks.syncronize_all')"
            value="all"
            v-model="task.foldersynchronization"
          />
          <cv-radio-button
            :name="'radio-group-foldersynchronization'"
            :label="$t('tasks.synchronize_only_INBOX')"
            value="inbox"
            v-model="task.foldersynchronization"
          />
          <cv-radio-button
            :name="'radio-group-foldersynchronization'"
            :label="$t('tasks.syncronize_with_exclusion')"
            value="exclusion"
            v-model="task.foldersynchronization"
          />
        </cv-radio-group>
        <template v-if="task.foldersynchronization == 'exclusion'">
          <cv-text-area
            class="mg-left"
            :label="$t('tasks.exclude_folder')"
            v-model.trim="task.exclude"
            ref="exclude"
            :invalid-message="$t(error.exclude)"
            :placeholder="$t('tasks.write_one_exclusion_per_line')"
            :helper-text="$t('tasks.start_by^_and_end_by$')"
          >
          </cv-text-area>
        </template>

        <span class="mg-bottom">
          {{ $t("tasks.remove_mails") }}
          <cv-tooltip
            alignment="start"
            direction="bottom"
            :tip="$t('tasks.imapsync_removal_explanation')"
            class="info mg-bottom"
          >
          </cv-tooltip>
        </span>
        <cv-radio-group vertical class="mg-bottom mg-left">
          <cv-radio-button
            :name="'radio-group-delete_local'"
            :label="$t('tasks.no_deletion')"
            value="no_delete"
            v-model="task.delete"
          />

          <cv-radio-button
            :name="'radio-group-delete_local'"
            :label="$t('tasks.delete_on_remote')"
            value="delete_remote"
            v-model="task.delete"
          />
          <!-- <cv-radio-button
            :name="'radio-group-delete_remote'"
            :label="$t('tasks.delete_on_local')"
            value="delete_local"
            v-model="task.delete"
          /> -->
        </cv-radio-group>
        <cv-dropdown
          :light="true"
          class="max-dropdown-width"
          :value="task.cron"
          v-model="task.cron"
          :up="false"
          :inline="false"
          :helper-text="$t('tasks.set_when_you_want_the_task_start')"
          :hide-selected="false"
          :label="$t('tasks.select_your_cron')"
        >
          <cv-dropdown-item selected value="">{{
            $t("tasks.no_cron")
          }}</cv-dropdown-item>
          <cv-dropdown-item value="5m">{{
            $t("tasks.every_minutes", { num: 5 })
          }}</cv-dropdown-item>
          <cv-dropdown-item value="10m">{{
            $t("tasks.every_minutes", { num: 10 })
          }}</cv-dropdown-item>
          <cv-dropdown-item value="15m">{{
            $t("tasks.every_minutes", { num: 15 })
          }}</cv-dropdown-item>
          <cv-dropdown-item value="30m">{{
            $t("tasks.every_minutes", { num: 30 })
          }}</cv-dropdown-item>
          <cv-dropdown-item value="45m">{{
            $t("tasks.every_minutes", { num: 45 })
          }}</cv-dropdown-item>
          <cv-dropdown-item value="1h">{{
            $t("tasks.every_minutes", { num: 60 })
          }}</cv-dropdown-item>
        </cv-dropdown>
      </cv-form>
      <cv-row v-if="error.createTask">
        <cv-column>
          <NsInlineNotification
            kind="error"
            :title="$t('tasks.create_task')"
            :description="error.createTask"
            :showCloseButton="false"
          />
        </cv-column>
      </cv-row>
    </template>
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
      loading: {
        createTask: false,
      },
      error: {
        enabled_mailboxe: "",
        createTask: "",
        exclude: "",
        localuser: "",
        remoteusername: "",
        remotepassword: "",
        remotehostname: "",
        remoteport: "",
      },
    };
  },
  computed: {
    ...mapState(["instanceName", "core", "appName"]),
  },
  methods: {
    validateConfigureModule() {
      this.clearErrors(this);

      let isValidationOk = true;
      if (!this.task.localuser) {
        this.error.localuser = "common.required";

        if (isValidationOk) {
          this.focusElement("localuser");
        }
        isValidationOk = false;
      }
      if (!this.task.remoteusername) {
        this.error.remoteusername = "common.required";

        if (isValidationOk) {
          this.focusElement("remoteusername");
        }
        isValidationOk = false;
      }
      if (!this.task.remotepassword) {
        this.error.remotepassword = "common.required";

        if (isValidationOk) {
          this.focusElement("remotepassword");
        }
        isValidationOk = false;
      }
      if (!this.task.remotehostname) {
        this.error.remotehostname = "common.required";

        if (isValidationOk) {
          this.focusElement("remotehostname");
        }
        isValidationOk = false;
      }
      if (!this.task.remoteport) {
        this.error.remoteport = "common.required";

        if (isValidationOk) {
          this.focusElement("remoteport");
        }
        isValidationOk = false;
      }
      if (
        this.task.foldersynchronization == "exclusion" &&
        !this.task.exclude
      ) {
        this.error.exclude = "common.required";

        if (isValidationOk) {
          this.focusElement("exclude");
        }
        isValidationOk = false;
      }
      return isValidationOk;
    },
    createTaskValidationFailed(validationErrors) {
      this.loading.createTask = false;
      let focusAlreadySet = false;
      for (const validationError of validationErrors) {
        const param = validationError.parameter;
        // set i18n error message
        this.error[param] = this.$t("tasks." + validationError.error);
        if (!focusAlreadySet) {
          this.focusElement(param);
          focusAlreadySet = true;
        }
      }
    },
    async createTask() {
      const isValidationOk = this.validateConfigureModule();
      if (!isValidationOk) {
        return;
      }
      this.loading.createTask = true;
      this.error.createTask = false;
      const taskAction = "create-task";
      const eventId = this.getUuid();
      // register to task error
      this.core.$root.$once(
        `${taskAction}-aborted-${eventId}`,
        this.setCreateTaskAborted
      );
      // register to task validation
      this.core.$root.$once(
        `${taskAction}-validation-failed-${eventId}`,
        this.createTaskValidationFailed
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
            task_id: this.task.task_id,
            localuser: this.task.localuser,
            remotehostname: this.task.remotehostname,
            remotepassword: this.task.remotepassword,
            remoteport: Number(this.task.remoteport),
            remoteusername: this.task.remoteusername,
            security: this.task.security,
            delete_local: this.task.delete === "delete_local" ? true : false,
            delete_remote: this.task.delete === "delete_remote" ? true : false,
            exclude: this.task.exclude
              .split("\n")
              .map((item) => item.trim())
              .join(","),
            cron: this.task.cron,
            foldersynchronization: this.task.foldersynchronization,
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
        this.error.createTask = this.getErrorMessage(err);
        this.loading.createTask = false;
        return;
      }
    },
    setCreateTaskAborted(taskResult, taskContext) {
      console.error(`${taskContext.action} aborted`, taskResult);
      this.error.createTask = this.$t("error.generic_error");
      this.loading.createTask = false;
    },
    setCreateTaskCompleted() {
      this.loading.createTask = false;
      this.clearErrors();
      this.$emit("hide");
      this.$emit("reloadtasks");
    },
    onModalHidden() {
      this.clearErrors();
      this.$emit("hide");
    },
  },
};
</script>
<style scoped lang="scss">
@import "../styles/carbon-utils";
.mg-bottom {
  margin-bottom: 1rem !important;
}
.mg-left {
  margin-left: 1rem !important;
}
.max-dropdown-width {
  max-width: 38rem;
}
</style>
