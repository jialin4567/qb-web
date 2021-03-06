<template>
    <v-dialog :value="value" @input="$emit('input', $event)" scrollable :fullscreen="phoneLayout" :width="dialogWidth">
        <v-card>
            <v-card-title
                    class="headline grey lighten-4"
            >
                <v-icon class="mr-2">mdi-delta</v-icon>
                <span>日志</span>
            </v-card-title>
            <v-card-text>
                <v-progress-linear
                        :indeterminate="true"
                        v-if="!logs"
                />
                <DynamicScroller
                        :items="reverseLogs"
                        :min-item-size="20"
                        class="logs caption"
                >
                    <template v-slot="{ item, index, active }">
                        <DynamicScrollerItem
                                :item="item"
                                :active="active"
                                :size-dependencies="[
          item.message,
        ]"
                                :data-index="item.id"
                        >
                            <div  :key="item.id" :class="item.type | typeColor">
                                {{index}}. [{{ item.type | formatType }} {{ item.timestamp / 1000 | formatTimestamp }}] <span
                                    v-html="item.message"/>
                            </div>

                        </DynamicScrollerItem>
                    </template>
                </DynamicScroller>

                <div ref="end"/>
            </v-card-text>
            <v-card-actions>
                <v-spacer/>
                <v-btn flat @click="closeDialog">关闭</v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script lang="ts">
    import Vue from "vue";
    import {api} from "@/Api";
    import {sleep} from "@/utils";
    import Taskable from "@/mixins/taskable";

    export default Vue.extend({
        mixins: [Taskable],

        props: {
            value: Boolean,
        },
        data() {
            return {
                logs: [],
            };
        },
        components: {
        },
        filters: {
            formatType(type: number) {
                const map: any = {
                    1: "N",
                    2: "I",
                    4: "W",
                    8: "C",
                };
                return map[type];
            },
            typeColor(type: number) {
                const map: any = {
                    1: "secondary--text",
                    2: "info--text",
                    4: "warn--text",
                    8: "error--text",
                };
                return map[type];
            },
        },
        computed: {
            dialogWidth() {
                return this.$vuetify.breakpoint.smAndDown ? "100%" : "70%";
            },
            phoneLayout() {
                return this.$vuetify.breakpoint.xsOnly;
            },
            reverseLogs(state){
                return _.reverse(_.clone(state.logs));
            }
        },
        methods: {
            closeDialog() {
                this.$emit("input", false);
            },
            async getLogs() {
                const lastId = this.logs.length ? this.logs[this.logs.length - 1]["id"] : -1;
                const logs = await api.getLogs(lastId);

                if (this.destory) {
                    return;
                }

                if (logs.length) {
                    this.logs = this.logs.concat(logs);

                    await this.$nextTick();

                    this.$refs.end.scrollIntoView();
                }

                this.task = setTimeout(this.getLogs, 2000);
            },
        },
        async created() {
            await this.getLogs();
        },
    });
</script>

<style lang="scss" scoped>
    .logs {
        font-family: monospace;

        li:not(:last-child) {
            line-height: 1.4em;
        }
    }
</style>
