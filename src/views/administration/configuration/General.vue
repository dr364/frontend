<template>
  <b-card no-body :header="header">
    <b-card-body>
      <b-validated-input-group-form-input
        id="base_url"
        :label="$t('admin.base_url')"
        input-group-size="mb-3"
        rules="required"
        type="url"
        v-model="baseUrl"
        tooltip="This URL is used to construct links back to Dependency-Track from external systems."
      />
      <c-switch id="isBadgesEnabled" color="primary" v-model="isBadgesEnabled" label v-bind="labelIcon" />{{$t('admin.enable_svg_badge')}}
      <b-validated-input-group-form-input
        id="cache_period"
        :label="$t('admin.cache_period')"
        input-group-size="mb-3"
        rules="required"
        type="number"
        v-model="cachePeriod"
        lazy="true"
        tooltip="Set the length of time to cache analyzer results. Analyzer requests will use the cache for requests inside the cache period."
      />
      <b-validated-input-group-form-input
        id="mirror_period"
        :label="$t('admin.mirror_period')"
        input-group-size="mb-3"
        rules="required"
        type="number"
        v-model="mirrorSyncPeriod"
        lazy="true"
        tooltip="Set the cadence for syncing new vulnerability reports from each of the analyzer databases."
      />
      <b-validated-input-group-form-input
        id="metrics_period"
        :label="$t('admin.metrics_period')"
        input-group-size="mb-3"
        rules="required"
        type="number"
        v-model="metricsUpdatePeriod"
        lazy="true"
        tooltip="Set the refresh cadence for metrics dashboards across the portfolio."
      />
    </b-card-body>
    <b-card-footer>
      <b-button variant="outline-primary" class="px-4" @click="saveChanges">{{ $t('message.update') }}</b-button>
    </b-card-footer>
  </b-card>
</template>

<script>
  import { Switch as cSwitch } from '@coreui/vue';
  import { ValidationObserver } from 'vee-validate';
  import BValidatedInputGroupFormInput from '../../../forms/BValidatedInputGroupFormInput'
  import common from "../../../shared/common";
  import configPropertyMixin from "../mixins/configPropertyMixin";

  export default {
    mixins: [configPropertyMixin],
    props: {
      header: String
    },
    components: {
      cSwitch,
      ValidationObserver,
      BValidatedInputGroupFormInput
    },
    data() {
      return {
        baseUrl: "",
        cachePeriod: 86400,
        mirrorSyncPeriod: 86400,
        metricsUpdatePeriod: 3600,
        isBadgesEnabled: false,
        labelIcon: {
          dataOn: '\u2713',
          dataOff: '\u2715'
        },
      }
    },
    methods: {
      saveChanges: function() {
        this.updateConfigProperties([
          {groupName: 'general', propertyName: 'base.url', propertyValue: this.baseUrl},
          {groupName: 'general', propertyName: 'cache.period', propertyValue: this.cachePeriod * 1000},
          {groupName: 'general', propertyName: 'mirror.period', propertyValue: this.mirrorSyncPeriod * 1000},
          {groupName: 'general', propertyName: 'metrics.period', propertyValue: this.metricsUpdatePeriod * 1000},
          {groupName: 'general', propertyName: 'badge.enabled', propertyValue: this.isBadgesEnabled}
        ]);
      }
    },
    created () {
      this.axios.get(this.configUrl).then((response) => {
        let configItems = response.data.filter(function (item) { return item.groupName === "general" });
        for (let i=0; i<configItems.length; i++) {
          let item = configItems[i];
          switch (item.propertyName) {
            case "base.url":
              this.baseUrl = item.propertyValue; break;
            case "cache.period":
              this.cachePeriod = item.propertyValue / 1000; break;
            case "mirror.period":
              this.mirrorSyncPeriod = item.propertyValue / 1000; break;
            case "metrics.period":
              this.metricsUpdatePeriod = item.propertyValue / 1000; break;
            case "badge.enabled":
              this.isBadgesEnabled = common.toBoolean(item.propertyValue); break;
          }
        }
      });
    }
  }
</script>
