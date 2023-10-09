<!--
  Copyright (C) 2023 Nethesis S.r.l.
  SPDX-License-Identifier: GPL-3.0-or-later
-->
<template>
  <NsModal
    size="default"
    :visible="isShown"
    @modal-hidden="$emit('hide')"
    @primary-click="$emit('confirm')"
  >
    <template slot="title"
      >{{
        task.remoteusername === ""
          ? $t("tasks.create_task")
          : $t("tasks.edit_task")
      }}: {{ task.mailbox }}</template
    >
    <template slot="content">
      <cv-form>
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
    <template slot="secondary-button">{{ core.$t("common.cancel") }}</template>
    <template slot="primary-button">{{ core.$t("common.save") }}</template>
  </NsModal>
</template>

<script>
import { UtilService, IconService } from "@nethserver/ns8-ui-lib";
export default {
  name: "CreateOrEditTask",
  mixins: [UtilService, IconService],
  props: {
    isShown: Boolean,
    task: { type: [Object, null] },
    core: { type: Object },
  },
};
</script>

<style scoped lang="scss">
@import "../styles/carbon-utils";
</style>
