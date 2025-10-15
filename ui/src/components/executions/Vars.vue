<template>
    <el-table table-layout="auto" fixed :data="variables">
        <el-table-column prop="key" min-width="500" :label="$t(keyLabelTranslationKey)">
            <template #default="scope">
                <code class="key-col">{{ getHumanizeLabel(scope.row.key) }}</code>
            </template>
        </el-table-column>

        <el-table-column prop="value" :label="$t('value')">
            <template #default="scope">
                <template v-if="scope.row.date">
                    <date-ago :inverted="true" :date="scope.row.value" />
                </template>
                <template v-else-if="scope.row.subflow">
                    {{ scope.row.value }}
                    <sub-flow-link :execution-id="scope.row.value" />
                </template>
                <template v-else>
                    <var-value :execution="execution" :value="scope.row.value" />
                </template>
            </template>
        </el-table-column>
    </el-table>
</template>

<script>
    import Utils from "../../utils/utils";
    import VarValue from "./VarValue.vue";
    import DateAgo from "../../components/layout/DateAgo.vue";
    import SubFlowLink from "../flows/SubFlowLink.vue"
    import {mapState} from "vuex";

    export default {
        components: {
            DateAgo,
            VarValue,
            SubFlowLink
        },
        props: {
            data: {
                type: Object,
                required: true
            },
            keyLabelTranslationKey: {
                type: String,
                required: false,
                default: "name"
            }
        },
        computed: {
            ...mapState("execution", ["execution"]),
            variables() {
                return Utils.executionVars(this.data);
            },
        },
        methods: {
            getHumanizeLabel(key) {
                // Known trigger/display keys mapped to existing translation keys
                const knownMappings = {
                    id: "id",
                    key: "key",
                    type: "type",
                    cron: "cron",
                    description: "description",
                    namespace: "namespace",
                    flowId: "flow",
                    nextExecutionDate: "next execution date",
                    updatedDate: "updated date",
                    date: "date",
                    state: "state",
                    states: "state" // avoid missing i18n key; use singular label
                };

                // Handle null/undefined keys
                if (!key) {
                    return key;
                }

                // Extract the last part of the key path for matching
                const lastPath = String(key).split(".").pop();
                const translationKey = knownMappings[lastPath];

                // Only translate when we have a known mapping to prevent i18n missing-key warnings
                if (translationKey) {
                    try {
                        return this.$t(translationKey);
                    } catch (error) {
                        // Fallback to raw key if translation fails
                        console.warn(`Translation failed for key '${translationKey}':`, error);
                        return key;
                    }
                }
                
                // Return the raw key if no mapping exists
                return key;
            }
        },
    };
</script>
<style>
    .key-col {
        min-width: 200px;
    }
</style>