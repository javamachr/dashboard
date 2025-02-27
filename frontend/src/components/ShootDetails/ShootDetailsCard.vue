<!--
SPDX-FileCopyrightText: 2021 SAP SE or an SAP affiliate company and Gardener contributors

SPDX-License-Identifier: Apache-2.0
-->

<template>
  <v-card>
    <v-toolbar flat dense color="toolbar-background toolbar-title--text">
      <v-toolbar-title class="text-subtitle-1">Details</v-toolbar-title>
    </v-toolbar>
    <v-list>
      <v-list-item>
        <v-list-item-icon>
          <v-icon color="primary">mdi-information-outline</v-icon>
        </v-list-item-icon>
        <v-list-item-content>
          <v-list-item-subtitle>Name</v-list-item-subtitle>
          <v-list-item-title class="pt-1">
            {{shootName}}
          </v-list-item-title>
        </v-list-item-content>
        <v-list-item-action>
          <copy-btn :clipboard-text="shootName"></copy-btn>
        </v-list-item-action>
      </v-list-item>
      <template v-if="expirationTimestamp && !isShootMarkedForDeletion">
        <v-divider inset></v-divider>
        <v-list-item>
          <v-list-item-icon>
            <v-icon color="primary">mdi-clock-outline</v-icon>
          </v-list-item-icon>
          <v-list-item-content>
            <v-list-item-subtitle>Cluster Termination</v-list-item-subtitle>
            <v-list-item-title class="d-flex align-center pt-1">
              <v-icon
                v-if="!isSelfTerminationWarning"
                color="primary"
                left
                size="18"
              >mdi-information</v-icon>
              <v-icon
                v-else
                color="warning"
                left
                size="18"
              >mdi-alert-circle</v-icon>
              <span>{{selfTerminationMessage}}</span>
            </v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </template>
      <v-divider inset></v-divider>
      <v-list-item>
        <v-list-item-icon>
          <v-icon color="primary">mdi-cube-outline</v-icon>
        </v-list-item-icon>
        <v-list-item-content>
          <v-list-item-subtitle>Kubernetes Version</v-list-item-subtitle>
          <v-list-item-title class="pt-1">
            {{shootK8sVersion}}
          </v-list-item-title>
        </v-list-item-content>
        <v-list-item-action class="mx-0" v-if="!isShootMarkedForDeletion">
          <version-expiration-warning :shoot-item="shootItem" only-k8s-warnings></version-expiration-warning>
        </v-list-item-action>
        <v-list-item-action class="mx-0">
          <shoot-version :shoot-item="shootItem"></shoot-version>
        </v-list-item-action>
      </v-list-item>
      <v-divider inset></v-divider>
      <v-list-item>
        <v-list-item-icon>
          <v-icon color="primary">mdi-server</v-icon>
        </v-list-item-icon>
        <v-list-item-content>
          <v-list-item-subtitle>Worker Groups</v-list-item-subtitle>
          <v-list-item-title class="d-flex flex-wrap align-center pt-1">
            <worker-group
            class="mr-2 mb-2"
            v-for="workerGroup in shootWorkerGroups"
            :worker-group="workerGroup"
            :cloud-profile-name="shootCloudProfileName"
            :shoot-item="shootItem"
            :key="workerGroup.name"
            ></worker-group>
          </v-list-item-title>
        </v-list-item-content>
        <v-list-item-action class="mx-0" v-if="!isShootMarkedForDeletion">
          <version-expiration-warning :shoot-item="shootItem" onlyMachineImageWarnings></version-expiration-warning>
        </v-list-item-action>
        <v-list-item-action class="mx-0">
          <worker-configuration :shoot-item="shootItem"></worker-configuration>
        </v-list-item-action>
      </v-list-item>
      <v-divider inset></v-divider>
      <v-list-item>
        <v-list-item-icon>
          <v-icon color="primary">mdi-account-outline</v-icon>
        </v-list-item-icon>
        <v-list-item-content>
          <v-list-item-subtitle>Created by</v-list-item-subtitle>
          <v-list-item-title class="pt-1">
            <account-avatar :account-name="shootCreatedBy" mail-to></account-avatar>
          </v-list-item-title>
        </v-list-item-content>
      </v-list-item>
      <v-list-item>
        <v-list-item-icon/>
        <v-list-item-content class="pt-0">
          <v-list-item-subtitle>Created at</v-list-item-subtitle>
          <v-list-item-title class="pt-1">
            <v-tooltip top>
              <template v-slot:activator="{ on }">
                <span v-on="on">{{shootCreatedAt}}</span>
              </template>
              <time-string :date-time="shootMetadata.creationTimestamp" mode="past"></time-string>
            </v-tooltip>
          </v-list-item-title>
        </v-list-item-content>
      </v-list-item>
      <template v-if="!!shootPurpose">
        <v-divider inset></v-divider>
        <v-list-item>
          <v-list-item-icon>
            <v-icon color="primary">mdi-label-outline</v-icon>
          </v-list-item-icon>
          <v-list-item-content>
            <v-list-item-subtitle>Purpose</v-list-item-subtitle>
            <v-list-item-title class="pt-1">
              {{shootPurpose}}
            </v-list-item-title>
          </v-list-item-content>
          <v-list-item-action>
            <!-- the selectable purposes depend on the used secretbinding which the user needs to be able to read in order to properly show the purpose configuration dialog -->
            <purpose-configuration v-if="canGetSecrets" :shoot-item="shootItem"></purpose-configuration>
          </v-list-item-action>
        </v-list-item>
      </template>
      <template v-if="slaDescriptionHtml">
        <v-divider inset></v-divider>
        <v-list-item>
          <v-list-item-icon>
            <v-icon color="primary">mdi-file-document-outline</v-icon>
          </v-list-item-icon>
          <v-list-item-content>
            <v-list-item-subtitle>{{slaTitle}}</v-list-item-subtitle>
            <v-list-item-title class="pt-1 markdown" v-html="slaDescriptionHtml"/>
          </v-list-item-content>
        </v-list-item>
      </template>
      <template v-if="cfg.accessRestriction">
        <v-divider inset></v-divider>
        <v-list-item>
          <v-list-item-icon>
            <v-icon color="primary">mdi-earth</v-icon>
          </v-list-item-icon>
          <v-list-item-content>
            <v-list-item-subtitle>Access Restrictions</v-list-item-subtitle>
            <v-list-item-title v-if="shootSelectedAccessRestrictions.length" class="d-flex align-center pt-1 flex-wrap">
              <access-restriction-chips :selected-access-restrictions="shootSelectedAccessRestrictions"></access-restriction-chips>
            </v-list-item-title>
            <v-list-item-title v-else class="d-flex align-center pt-1">
              No access restrictions configured
            </v-list-item-title>
          </v-list-item-content>
          <v-list-item-action>
            <access-restrictions-configuration
              :shoot-item="shootItem"></access-restrictions-configuration>
          </v-list-item-action>
        </v-list-item>
      </template>
      <v-divider inset></v-divider>
      <v-list-item>
        <v-list-item-icon>
          <v-icon color="primary">mdi-puzzle</v-icon>
        </v-list-item-icon>
        <v-list-item-content>
          <v-list-item-subtitle>Add-ons <span class="text-caption">(not actively monitored and provided on a best-effort basis only)</span></v-list-item-subtitle>
          <v-list-item-title class="d-flex align-center pt-1">
            <template v-if="this.shootAddonNames.length">
              <v-chip v-for="(name, index) in this.shootAddonNames"
                :key="index"
                small
                outlined
                color="primary"
                class="mr-2"
              >
              {{name}}
              </v-chip>
            </template>
            <span v-else>No addons configured</span>
          </v-list-item-title>
        </v-list-item-content>
        <v-list-item-action>
          <addon-configuration :shoot-item="shootItem"></addon-configuration>
        </v-list-item-action>
      </v-list-item>
    </v-list>
  </v-card>
</template>

<script>
import { mapState, mapGetters } from 'vuex'
import filter from 'lodash/filter'
import map from 'lodash/map'

import AccessRestrictionChips from '@/components/ShootAccessRestrictions/AccessRestrictionChips'
import AccountAvatar from '@/components/AccountAvatar'
import TimeString from '@/components/TimeString'
import WorkerGroup from '@/components/ShootWorkers/WorkerGroup'
import WorkerConfiguration from '@/components/ShootWorkers/WorkerConfiguration'
import AccessRestrictionsConfiguration from '@/components/ShootAccessRestrictions/AccessRestrictionsConfiguration'
import PurposeConfiguration from '@/components/PurposeConfiguration'
import ShootVersion from '@/components/ShootVersion/ShootVersion'
import VersionExpirationWarning from '@/components/VersionExpirationWarning'
import AddonConfiguration from '@/components/ShootAddons/AddonConfiguration'
import CopyBtn from '@/components/CopyBtn'

import {
  isSelfTerminationWarning,
  isValidTerminationDate,
  getTimeStringTo,
  shootAddonList,
  transformHtml
} from '@/utils'

import { shootItem } from '@/mixins/shootItem'

export default {
  components: {
    AccessRestrictionChips,
    AccountAvatar,
    TimeString,
    WorkerGroup,
    WorkerConfiguration,
    AccessRestrictionsConfiguration,
    PurposeConfiguration,
    AddonConfiguration,
    ShootVersion,
    VersionExpirationWarning,
    CopyBtn
  },
  props: {
    shootItem: {
      type: Object
    }
  },
  mixins: [shootItem],
  computed: {
    ...mapState([
      'cfg'
    ]),
    ...mapGetters([
      'canGetSecrets'
    ]),
    expirationTimestamp () {
      return this.shootAnnotations['shoot.gardener.cloud/expiration-timestamp'] || this.shootAnnotations['shoot.garden.sapcloud.io/expirationTimestamp']
    },
    selfTerminationMessage () {
      if (this.isValidTerminationDate) {
        return `This cluster will self terminate ${getTimeStringTo(new Date(), new Date(this.expirationTimestamp))}`
      } else {
        return 'This cluster is about to self terminate'
      }
    },
    isSelfTerminationWarning () {
      return isSelfTerminationWarning(this.expirationTimestamp)
    },
    isValidTerminationDate () {
      return isValidTerminationDate(this.expirationTimestamp)
    },
    addon () {
      return (name) => {
        return this.shootAddons[name] || {}
      }
    },
    shootAddonNames () {
      return map(filter(shootAddonList, item => this.addon(item.name).enabled), 'title')
    },
    sla () {
      return this.cfg.sla || {}
    },
    slaDescriptionHtml () {
      return transformHtml(this.sla.description)
    },
    slaTitle () {
      return this.sla.title
    }
  }
}
</script>

<style lang="scss" scoped>

  .markdown {
    ::v-deep > p {
      margin: 0px;
    }
  }

</style>
