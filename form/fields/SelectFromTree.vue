<template>
    <div>
        <v-card class="" v-if="info">
            <slot name="header" v-bind:info="info">
            </slot>

            <v-card-text class="pa-0">
                <v-sheet class="py-2 px-4 green lighten-3">
                    <v-text-field v-model="tree_search" label="Поиск" dark flat dense solo-inverted hide-details clearable clear-icon="mdi-close-circle-outline"></v-text-field>
                </v-sheet>
                <v-card-text>
                    <v-treeview v-model="selected_tree_items" :items="tree_items" :search="tree_search" selectable open-on-click :open="opened" @input="changeSelect">
                         <template v-slot:label="{ item }">
                            <slot name="item" v-bind:info="item">
                                <span class="subtitle">{{ item.name }}</span>
                            </slot>
                         </template>
                    </v-treeview>
                </v-card-text>
            </v-card-text>

            <slot name="footer">
            </slot>
        </v-card>
    </div>
</template>

<script>
import Vue from "vue";

export default {
    components: {
    },
    props: {
        value: { required: true },
        tableName: { type: String, default: "" },
        tableCaption: { type: String, default: "" },
        afterReloadTable: { type: Function, default: null },
        expandFirstLevel: { type: Boolean, default: true },
        multiple: { type: Boolean, default: false },
    },
    watch: {
        tableName() {
            this.reloadTable();
        },
    },
    data() {
        return {
            opened: [],
            info: null,
            columns: [],
            tree_items: [],
            selected_tree_items: [],
            tree_search: "",
        };
    },
    mounted() {
        this.reloadTable();
    },
    computed: {},
    methods: {
        showLoader(display) {
            this.$store.commit("SHOW_LOADER", display);
        },

        reloadTable() {
            if (!this.tableName) return;

            this.showLoader(true);
            this.$api("table", "tree", this.tableName)
                .then((response) => {
                    if (this.afterReloadTable) response = this.afterReloadTable(response);
                    this.showLoader(false);
                    this.tree_items = response.rows;
                    this.selected_tree_items = this.value;
                    this.info = response.info;
                    this.columns = response.info.columns;
                    this.tree_items.forEach((e) => { this.opened.push(e.id) });
                })
                .catch(() => {
                    this.showLoader(false);
                });
        },

        changeSelect(vals) {
            if (!this.multiple && vals.length>1) {
                vals = [ vals[vals.length-1] ];
                this.selected_tree_items = vals;
            }

            const sels = [];
            const cb = (e) => {
                if (vals.includes(e.id)) sels.push(e);
                if (e.children) e.children.forEach(cb);
            };
            this.tree_items.forEach(cb);

            this.$emit('input', vals);
            this.$emit('select', sels );
        }

    },//methods
};
</script>

<style></style>
