<!--
SPDX-FileCopyrightText: 2021 SAP SE or an SAP affiliate company and Gardener contributors

SPDX-License-Identifier: Apache-2.0
 -->
<template>
  <v-container fluid class="px-6">
    <v-row class="d-flex">
      <v-col cols="12" md="6">
        <shoot-details-card :shoot-item="shootItem"></shoot-details-card>
        <custom-fields-card :custom-fields="customFields" class="mt-4"></custom-fields-card>
        <shoot-infrastructure-card :shoot-item="shootItem" class="mt-4"></shoot-infrastructure-card>
        <shoot-external-tools-card :shoot-item="shootItem" class="mt-4"></shoot-external-tools-card>
        <shoot-lifecycle-card ref="shootLifecycle" :shoot-item="shootItem" class="mt-4"></shoot-lifecycle-card>
      </v-col>
      <v-col cols="12" md="6">
        <v-card v-if="canGetSecrets" class="mb-4">
          <v-toolbar flat dense color="toolbar-background toolbar-title--text">
            <v-toolbar-title class="text-subtitle-1">Access</v-toolbar-title>
          </v-toolbar>
          <shoot-access-card :shoot-item="shootItem" @add-terminal-shortcut="onAddTerminalShortcut"></shoot-access-card>
        </v-card>
        <shoot-monitoring-card :shoot-item="shootItem"></shoot-monitoring-card>
        <tickets-card :tickets="tickets" :shoot-item="shootItem" class="mt-4"></tickets-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import { mapGetters } from 'vuex'
import filter from 'lodash/filter'
import get from 'lodash/get'
import map from 'lodash/map'

import ShootDetailsCard from '@/components/ShootDetails/ShootDetailsCard'
import CustomFieldsCard from '@/components/ShootDetails/CustomFieldsCard'
import ShootExternalToolsCard from '@/components/ShootDetails/ShootExternalToolsCard'
import ShootInfrastructureCard from '@/components/ShootDetails/ShootInfrastructureCard'
import ShootLifecycleCard from '@/components/ShootDetails/ShootLifecycleCard'
import ShootMonitoringCard from '@/components/ShootDetails/ShootMonitoringCard'
import TicketsCard from '@/components/TicketsCard'

import { shootItem } from '@/mixins/shootItem'

import 'codemirror/mode/yaml/yaml.js'

const ShootAccessCard = () => import('@/components/ShootDetails/ShootAccessCard')

export default {
  name: 'shoot-details',
  components: {
    ShootDetailsCard,
    CustomFieldsCard,
    ShootInfrastructureCard,
    ShootLifecycleCard,
    ShootAccessCard,
    TicketsCard,
    ShootMonitoringCard,
    ShootExternalToolsCard
  },
  mixins: [shootItem],
  props: {
    shootItem: {
      type: Object
    }
  },
  computed: {
    ...mapGetters([
      'ticketsByNamespaceAndName',
      'canGetSecrets',
      'shootCustomFieldList'
    ]),
    info () {
      return get(this, 'shootItem.info', {})
    },
    seedInfo () {
      return get(this, 'shootItem.seedInfo', {})
    },
    tickets () {
      const namespace = this.shootNamespace
      const name = this.shootName
      return this.ticketsByNamespaceAndName({ name, namespace })
    },
    customFields () {
      const customFields = filter(this.shootCustomFieldList, ['showDetails', true])
      return map(customFields, ({ name, path, icon, tooltip, defaultValue }) => ({
        name,
        path,
        icon,
        tooltip,
        defaultValue,
        value: get(this.shootItem, path)
      }))
    }
  },
  methods: {
    onAddTerminalShortcut (shortcut) {
      this.$emit('add-terminal-shortcut', shortcut)
    }
  },
  mounted () {
    if (get(this.$route, 'name') === 'ShootItemHibernationSettings') {
      this.$refs.shootLifecycle.showHibernationConfigurationDialog()
    }
  }
}
</script>

<style lang="scss" scoped>
  .subheading.v-card__title {
    line-height: 10px;
  }
</style>
