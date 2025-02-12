<template>
  <actionable-list-group-item :delete-icon="true" v-on:actionClicked="removeCondition()">
    <b-row>
      <b-col md="4" lg="3">
        <b-input-group-form-select id="input-subject" required="true"
                                   v-on:change="subjectChanged" v-model="subject" :options="subjects" />
      </b-col>
      <b-col md="3" lg="2">
        <b-input-group-form-select id="input-operator" required="true"
                                   v-model="operator" :options="operators" />
      </b-col>
      <b-col md="5" lg="5">
        <b-input-group-form-select v-if="subject !== 'COORDINATES' && isSubjectSelectable" id="input-value" required="true"
                                   v-on:change="saveCondition" v-model="value" :options="possibleValues" />

        <b-input-group-form-input v-else-if="subject !== 'COORDINATES' && !isSubjectSelectable" id="input-value" required="true" type="text" v-model="value" lazy="true"
                                  v-debounce:750ms="saveCondition" :debounce-events="'keyup'" />

        <b-input-group v-else-if="subject === 'COORDINATES'">
          <b-form-input id="input-value-coordinates-group" :placeholder="$t('message.group')" type="text" v-model="coordinatesGroup" v-debounce:750ms="saveCondition" :debounce-events="'keyup'"></b-form-input>
          <b-form-input id="input-value-coordinates-name" :placeholder="$t('message.name')" type="text" v-model="coordinatesName" v-debounce:750ms="saveCondition" :debounce-events="'keyup'"></b-form-input>
          <b-form-input id="input-value-coordinates-version" :placeholder="$t('message.version')" type="text" v-model="coordinatesVersion" v-debounce:750ms="saveCondition" :debounce-events="'keyup'"></b-form-input>
          <b-tooltip target="input-value-coordinates-version" triggers="hover focus">{{ $t('message.coordinates_version_tooltip') }}</b-tooltip>
        </b-input-group>

      </b-col>
      <b-col md="0" lg="2">
      </b-col>
    </b-row>
  </actionable-list-group-item>
</template>

<script>
  import ActionableListGroupItem from "../components/ActionableListGroupItem";
  import BInputGroupFormSelect from "../../forms/BInputGroupFormSelect";
  import BInputGroupFormInput from "../../forms/BInputGroupFormInput";
  import common from "../../shared/common";

  export default {
    props: {
      policy: Object,
      condition: Object
    },
    components: {
      ActionableListGroupItem,
      BInputGroupFormSelect,
      BInputGroupFormInput
    },
    created() {
      if (this.condition) {
        this.subject = this.condition.subject;
        this.subjectChanged();
        this.operator = this.condition.operator;
        this.value = this.condition.value;
      }
    },
    data() {
      return {
        subject: null,
        operator: null,
        value: null,
        coordinatesGroup: null,
        coordinatesName: null,
        coordinatesVersion: null,
        subjects: [
          //{value: 'AGE', text: this.$t('message.age')},
          //{value: 'ANALYZER', text: this.$t('message.analyzer')},
          //{value: 'BOM', text: this.$t('message.bom')},
          {value: 'SEVERITY', text: this.$t('message.severity')},
          {value: 'COORDINATES', text: this.$t('message.coordinates')},
          {value: 'LICENSE', text: this.$t('message.license')},
          {value: 'LICENSE_GROUP', text: this.$t('message.license_group')},
          {value: 'PACKAGE_URL', text: this.$t('message.package_url')},
          {value: 'CPE', text: this.$t('message.cpe_full')},
          {value: 'SWID_TAGID', text: this.$t('message.swid_tagid')},
          {value: 'VERSION', text: this.$t('message.version')},
          {value: 'COMPONENT_HASH', text: this.$t('message.component_hash')},
          {value: 'CWE', text: this.$t('message.cwe_full')}
        ],
        objectOperators: [
          {value: 'IS', text: this.$t('operator.is')},
          {value: 'IS_NOT', text: this.$t('operator.is_not')}
        ],
        regexOperators: [
          {value: 'MATCHES', text: this.$t('operator.matches')},
          {value: 'NO_MATCH', text: this.$t('operator.no_match')}
        ],
        numericOperators: [
          {value: 'NUMERIC_GREATER_THAN', text: '>'},
          {value: 'NUMERIC_LESS_THAN', text: '<'},
          {value: 'NUMERIC_EQUAL', text: '='},
          {value: 'NUMERIC_NOT_EQUAL', text: '≠'},
          {value: 'NUMERIC_GREATER_THAN_OR_EQUAL', text: '≥'},
          {value: 'NUMERIC_LESSER_THAN_OR_EQUAL', text: '≤'}
        ],
        hashAlgorithms: [
          {value: 'MD5', text: this.$t('hashes.md5')},
          {value: 'SHA-1', text: this.$t('hashes.sha_1')},
          {value: 'SHA-256', text: this.$t('hashes.sha_256')},
          {value: 'SHA-384', text: this.$t('hashes.sha_384')},
          {value: 'SHA-512', text: this.$t('hashes.sha_512')},
          {value: 'SHA3-256', text: this.$t('hashes.sha3_256')},
          {value: 'SHA3-384', text: this.$t('hashes.sha3_384')},
          {value: 'SHA3-512', text: this.$t('hashes.sha3_512')},
          {value: 'BLAKE2b-256', text: this.$t('hashes.blake_256')},
          {value: 'BLAKE2b-384', text: this.$t('hashes.blake_384')},
          {value: 'BLAKE2b-512', text: this.$t('hashes.blake_512')},
          {value: 'BLAKE3', text: this.$t('hashes.blake3')}
        ],
        listOperators: [
          {value: 'CONTAINS_ANY', text: this.$t('operator.contains_any')},
          {value: 'CONTAINS_ALL', text: this.$t('operator.contains_all')}
        ],
        operators: [],
        possibleValues: []
      }
    },
    computed: {
      isSubjectSelectable: function() {
        switch(this.subject) {
          case 'AGE':
            return false;
          case 'ANALYZER':
            return true;
          case 'BOM':
            return true;
          case 'SEVERITY':
            return true;
          case 'COORDINATES':
            return false;
          case 'LICENSE':
            return true;
          case 'LICENSE_GROUP':
            return true;
          case 'PACKAGE_URL':
            return false;
          case 'CPE':
            return false;
          case 'SWID_TAGID':
            return false;
          case 'VERSION':
            return false;
          case 'COMPONENT_HASH':
            return false;
          case 'CWE':
            return false;
          default:
            return false;
        }
      }
    },
    beforeMount() {
      if (this.subject === "COORDINATES") {
        let v = JSON.parse(this.value);
        if (v) {
          this.coordinatesGroup = v.group;
          this.coordinatesName = v.name;
          this.coordinatesVersion = v.version;
        }
      }
    },
    methods: {
      subjectChanged: function() {
        switch(this.subject) {
          case 'AGE':
            this.operators = this.numericOperators;
            break;
          case 'ANALYZER':
            this.operators = this.objectOperators;
            break;
          case 'BOM':
            this.operators = this.objectOperators;
            break;
          case 'SEVERITY':
            this.operators = this.objectOperators;
            this.populateSeverity();
            break;
          case 'COORDINATES':
            this.operators = this.regexOperators;
            break;
          case 'LICENSE':
            this.operators = this.objectOperators;
            this.retrieveLicenses();
            break;
          case 'LICENSE_GROUP':
            this.operators = this.objectOperators;
            this.retrieveLicenseGroups();
            break;
          case 'PACKAGE_URL':
            this.operators = this.regexOperators;
            break;
          case 'CPE':
            this.operators = this.regexOperators;
            break;
          case 'SWID_TAGID':
            this.operators = this.regexOperators;
            break;
          case 'VERSION':
            this.operators = this.numericOperators;
            break;
          case 'COMPONENT_HASH':
            this.operators = this.hashAlgorithms;
            break;
          case 'CWE':
            this.operators = this.listOperators;
            break;
          default:
            this.operators = [];
        }
        this.saveCondition();
      },
      createDynamicValue: function() {
        if (this.subject === "COORDINATES") {
          return JSON.stringify({
            group: common.trimToNull(this.coordinatesGroup),
            name: common.trimToNull(this.coordinatesName),
            version: common.trimToNull(this.coordinatesVersion)
          });
        } else if (this.subject === "COMPONENT_HASH") {
          return JSON.stringify({
            algorithm: common.trimToNull(this.operator),
            value: common.trimToNull(this.value)
          });
        } else {
          return this.value;
        }
      },
      saveCondition: function() {
        let dynamicValue = this.createDynamicValue();
        if (!this.subject || !this.operator || !dynamicValue) {
          return;
        }
        if (this.condition.uuid) {
          let url = `${this.$api.BASE_URL}/${this.$api.URL_POLICY}/condition`;
          this.axios.post(url, {
            uuid: this.condition.uuid,
            subject: this.subject,
            operator: this.subject === 'COMPONENT_HASH' ? 'IS' : this.operator,
            value: dynamicValue
          }).then((response) => {
            this.condition = response.data;
            this.$toastr.s(this.$t('message.updated'));
          }).catch((error) => {
            this.$toastr.w(this.$t('condition.unsuccessful_action'));
          });
        } else {
          let url = `${this.$api.BASE_URL}/${this.$api.URL_POLICY}/${this.policy.uuid}/condition`;
          this.axios.put(url, {
            subject: this.subject,
            operator: this.subject === 'COMPONENT_HASH' ? 'IS' : this.operator,
            value: dynamicValue
          }).then((response) => {
            this.condition = response.data;
            this.$toastr.s(this.$t('message.updated'));
          }).catch((error) => {
            this.$toastr.w(this.$t('condition.unsuccessful_action'));
          });
        }
      },
      removeCondition: function() {
        if (this.condition && this.condition.uuid) {
          let url = `${this.$api.BASE_URL}/${this.$api.URL_POLICY}/condition/${this.condition.uuid}`;
          this.axios.delete(url).then((response) => {
            this.condition = response.data;
            this.$toastr.s(this.$t('message.condition_deleted'));
            this.$emit('conditionRemoved');
          }).catch((error) => {
            this.$toastr.w(this.$t('condition.unsuccessful_action'));
          });
        } else {
          this.$emit('conditionRemoved');
        }
      },
      retrieveLicenseGroups: function() {
        this.axios.get(`${this.$api.BASE_URL}/${this.$api.URL_LICENSE_GROUP}?limit=9999`
        ).then((response) => {
          let vals = [];
          for (let i=0; i<response.data.length; i++) {
            let object = response.data[i];
            vals.push( {value: object.uuid, text: object.name} )
          }
          this.possibleValues = vals;
        }).catch((error) => {
          if (error.response.status === 304) {
            //this.$toastr.w(this.$t('condition.unsuccessful_action'));
          } else {
            this.$toastr.w(this.$t('condition.unsuccessful_action'));
          }
        });
      },
      retrieveLicenses: function() {
        this.axios.get(`${this.$api.BASE_URL}/${this.$api.URL_LICENSE_CONCISE}?limit=9999`
        ).then((response) => {
          let vals = [];
          vals.push( {value: "unresolved", text: "unresolved"} )
          for (let i=0; i<response.data.length; i++) {
            let object = response.data[i];
            vals.push( {value: object.uuid, text: object.name} )
          }
          this.possibleValues = vals;
        }).catch((error) => {
          if (error.response.status === 304) {
            //this.$toastr.w(this.$t('condition.unsuccessful_action'));
          } else {
            this.$toastr.w(this.$t('condition.unsuccessful_action'));
          }
        });
      },
      populateSeverity: function () {
        this.possibleValues = [
          {value: "CRITICAL", text: this.$t('severity.critical')},
          {value: "HIGH", text: this.$t('severity.high')},
          {value: "MEDIUM", text: this.$t('severity.medium')},
          {value: "LOW", text: this.$t('severity.low')},
          {value: "INFO", text: this.$t('severity.info')},
          {value: "UNASSIGNED", text: this.$t('severity.unassigned')}
        ];
      }
    }
  }
</script>
