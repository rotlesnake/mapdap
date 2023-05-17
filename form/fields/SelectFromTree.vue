<template>
    <div>
        <v-card class="" v-if="info">
            <slot name="header" v-bind:info="info">
            </slot>

            <v-card-text class="pa-0">
                <v-sheet class="py-2 px-4 green lighten-3">
                    <v-card-title class="pa-0">
                      <v-text-field v-model="tree_search" label="Поиск" dark flat dense solo-inverted hide-details clearable clear-icon="mdi-close-circle-outline"></v-text-field>
                      <div class="ml-4">
                          <v-btn @click="openAll" fab small><v-icon>mdi-expand-all</v-icon></v-btn>
                          <v-btn @click="closeAll" fab small><v-icon>mdi-collapse-all</v-icon></v-btn>
                      </div>
                    </v-card-title>
                </v-sheet>
                <v-card-text>
                    <v-treeview v-model="selected_tree_items" :items="tree_items" :search="tree_search" :filter="filter_search" selectable open-on-click :open="opened" @input="changeSelect">
                         <template v-slot:label="{ item }">
                            <slot name="item" v-bind:info="item">
                                <div v-if="showID" class="subtitle" style="line-height: 34px" @click="selected_tree_items.push(item.id)">{{ item.id }}. {{ calcFieldName(item) }}</div>
                                <div v-else class="subtitle" style="line-height: 34px" @click="selected_tree_items.push(item.id)">{{ calcFieldName(item) }}</div>
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
        fieldName: { type: String, default: "name" },
        fieldAltName: { type: String, default: "" },
        tableCaption: { type: String, default: "" },
        afterReloadTable: { type: Function, default: null },
        expandFirstLevel: { type: Boolean, default: true },
        multiple: { type: Boolean, default: false },
        showID: { type: Boolean, default: false },
    },
    watch: {
        value() {
            this.selected_tree_items = Array.isArray(this.value) ? this.value : [this.value];
        },
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
    computed: {
        filter_search() {
            return (item, search, textKey) => {
                search = String(search).toLowerCase();
                let fields = ["name"];
                this.fieldName.replace(/\[(.*?)\]/gi, (match, name) => {
                    fields.push(name);
                });
                this.fieldAltName.replace(/\{\{(.*?)\}\}/gi, (match, name) => {
                    fields.push(name);
                });
                let rez = false;
                fields.forEach(e=>{
                    if (String(item[e]).toLowerCase().indexOf(search) > -1) rez = true; 
                });

                return rez;
            };
        },
    },
    methods: {
        showLoader(display) {
            this.$store.commit("SHOW_LOADER", display);
        },

        calcFieldName(item) {
            let text = "";
            let text2 = "";

            if (this.fieldName.indexOf("[") == -1) {
                text = item[this.fieldName];
            } else {
                text = this.fieldName.replace(/\[(.*?)\]/gi, (match, name) => item[name] );
                text = text.replace(/\{\{(.*?)\}\}/gi, (match, name) => item[name]);
            }

            if (this.fieldAltName.indexOf("[") == -1) {
                text2 = item[this.fieldAltName];
            } else {
                text2 = this.fieldAltName.replace(/\[(.*?)\]/gi, (match, name) => item[name] );
                text2 = text2.replace(/\{\{(.*?)\}\}/gi, (match, name) => item[name]);
            }

            text = String(text).trim();
            text2 = String(text2).trim();

            return text.length>0 ? text : text2;
        },

        reloadTable() {
            if (!this.tableName) return;

            this.showLoader(true);
            this.$api("table", "tree", this.tableName)
                .then((response) => {
                    if (this.afterReloadTable) response = this.afterReloadTable(response);
                    this.showLoader(false);
                    this.tree_items = response.rows;
                    this.selected_tree_items = Array.isArray(this.value) ? this.value : [this.value];
                    this.info = response.info;
                    this.columns = response.info.columns;
                })
                .catch(() => {
                    this.showLoader(false);
                });
        },

        openAll(){ this.tree_items.forEach((e) => { this.opened.push(e.id) }); },
        closeAll(){ this.opened = [] },

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
