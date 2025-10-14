<template>
    <el-table stripe table-layout="auto" fixed :data="variables">
        <el-table-column prop="key" rowspan="3" :label="$t('name')">
            <template #default="scope">
                <code>{{ getHumanizeLabel(scope.row.key) }}</code>
            </template>
        </el-table-column>

        <el-table-column prop="value" :label="$t('value')">
            <template #default="scope">
                <template v-if="scope.row.key === 'description'">
                    <markdown :source="scope.row.value" />
                </template>
                <template v-else-if="scope.row.key === 'cron'">
                    <cron :cron-expression="scope.row.value" />
                </template>
                <template v-else-if="scope.row.key === 'key'">
                    {{ scope.row.value }}
                    <el-button @click="emit('on-copy', null)">
                        {{ $t('copy url') }}
                    </el-button>
                </template>
                <template v-else>
                    <var-value :value="scope.row.value" :execution="execution" />
                </template>
            </template>
        </el-table-column>
    </el-table>
</template>

<script>
    import Utils from "../../utils/utils";
    import VarValue from "../executions/VarValue.vue";
    import Markdown from "../layout/Markdown.vue";
    import Cron from "../layout/Cron.vue";

    export default {
        emits: ["on-copy"],
        components: {
            VarValue,
            Markdown,
            Cron
        },
        props: {
            data: {
                type: Object,
                required: true
            },
            execution: {
                type: Object,
                required: false,
                default: undefined
            }
        },
        computed: {
            variables() {
                return Utils.executionVars(this.data);
            },
        },
        methods: {
            emit(type, event) {
                this.$emit(type, event);
            },
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

                const lastPath = String(key).split(".").pop();
                const translationKey = knownMappings[lastPath] || null;

                // Only translate when we have a known mapping to prevent i18n missing-key warnings
                return translationKey ? this.$t(translationKey) : key;
            }
        }
    };
</script>

<style lang="scss" scoped>
    :deep(.markdown) {
        p {
            margin-bottom: auto;
        }
    }
</style>