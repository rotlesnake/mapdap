<template>
    <div>
        <v-select
            @mouseup="open()"
            clearable
            @change="onchange()"
            readonly
            @click:clear.prevent="onClear"
            :items="items"
            v-model="values"
            :multiple="true"
            chips
            item-value="value"
            item-text="text"
            v-bind="opts"
            :rules="rules"
            :disabled="disabled"
        >
            <template v-slot:selection="{ item, index }">
                <v-chip :small="opts.dense">
                    <span>{{ item.text }}</span>
                </v-chip>
            </template>
        </v-select>

        <v-dialog v-model="display">
            <!-- TABLE !-->
            <v-card v-if="typeSelect == 'table'">
                <v-card-text class="pa-2" v-if="display && opts">
                    <mdp-table
                        ref="TableSelector"
                        v-model="values"
                        v-bind="opts.options || {}"
                        :tableCaption="opts.label"
                        :tableName="opts.table"
                        :multiple="opts.multiple"
                        @select="rowSelected"
                        @dblSelect="finishSelect()"
                    >
                        <template v-slot:after_footer_buttons>
                            <v-divider vertical class="mx-4"></v-divider>
                            <v-btn
                                elevation="4"
                                color="primary"
                                @click="finishSelect()"
                                :disabled="selected.length == 0"
                                >Выбрать</v-btn
                            >

                            <v-btn class="float-right mr-4" color="red" @click="display = false">Закрыть</v-btn>
                        </template>
                    </mdp-table>
                </v-card-text>
            </v-card>
            <!-- TABLE !-->

            <!-- TREE !-->
            <v-card v-if="typeSelect == 'tree'">
                <mdp-tree
                        ref="TableSelector"
                        v-model="values"
                        v-bind="opts.options || {}"
                        :tableCaption="opts.label"
                        :tableName="opts.table"
                        :multiple="opts.multiple"
                        @select="rowSelected"
                >
                    <template v-slot:header="{ info }">
                        <v-card-title class="pa-0 ma-0">
                            <v-toolbar dense color="primary" elevation="0">
                                <v-toolbar-title class="white--text">{{info.name}}</v-toolbar-title>
                                <v-spacer></v-spacer>
                                <v-btn icon dark @click="display = false"><v-icon>close</v-icon></v-btn>
                            </v-toolbar>
                        </v-card-title>
                     </template>

                     <template v-slot:footer>
                         <v-divider />
                         <v-card-actions class="pa-4">
                            <v-btn
                                elevation="4"
                                color="primary"
                                @click="finishSelect()"
                                :disabled="selected.length == 0"
                                >Выбрать</v-btn
                            >
                            <v-spacer />
                            <v-btn class="float-right mr-4" color="red" @click="display = false">Закрыть</v-btn>
                         </v-card-actions>
                     </template>
                </mdp-tree>
            </v-card>
            <!-- TREE !-->
        </v-dialog>
    </div>
</template>

<script>
import Vue from "vue";

export default {
    components: {
        "mdp-table": () => import("../../table/Table"),
        "mdp-tree": () => import("./SelectFromTree.vue"),
    },
    props: {
        row: { required: true },
        value: { required: true },
        items: { type: Array, default: () => [] },
        rules: { type: Array, default: () => [] },
        disabled: { default: false },
        options: { type: Object, default: null },
    },
    watch: {
        row() {
            this.refresh();
        },
        options() {
            this.refresh();
        },
        value() {
            this.refresh();
        },
        items() {
            this.refresh();
        },
    },
    data() {
        return {
            typeSelect: 'table',
            display: false,
            selected: [],
            opts: null,
            values: [],
            labels: "",
        };
    },
    mounted() {
        this.refresh();
    },
    computed: {},
    methods: {
        open(){ 
           if (this.disabled) return;
           this.display = true;
        },
        refresh() {
            this.opts = JSON.parse(JSON.stringify(this.options));

            this.typeSelect = "table";
            if (this.opts.typeSelect && this.opts.typeSelect=="tree") this.typeSelect = this.opts.typeSelect;

            if (this.opts.options) {
                //спецопции для таблицы
                if (this.opts.options.tableFilter) {
                    this.opts.options.tableFilter.forEach((e) => {
                        if (e.value.indexOf("[") > -1) {
                            let value = e.value.replace(/\[(.*?)\]/gi, (match, name) => {
                                if (!this.row[name]) return 0;
                                return this.row[name];
                            });
                            e.value = value;
                        }
                    });
                }
            }

            if (this.value) {
                this.values = this.value;
                if (typeof this.value != "object") this.values = [parseInt(this.value)];
            }
        },
        onchange() {
            //console.log(this.values);
        },
        onClear() {
            this.$emit("change", [], []);
            this.$nextTick().then(() => {
                this.display = false;
            });
        },
        rowSelected(rows) {
            this.selected = rows;
        },
        rowDblclick() {
            this.finishSelect();
        },
        onAddRow(row) {
            this.selected = [row];
            this.finishSelect();
        },
        finishSelect() {
                let items = [];
                this.values = [];
                for (let item of this.selected) {
                    let text = "";
                    if (this.opts.field.indexOf("[") == -1) {
                        text = item[this.opts.field];
                    } else {
                        text = this.opts.field.replace(/\[(.*?)\]/gi, (match, name) => item[name]);
                    }
                    items.push({
                        value: item.id,
                        text: text,
                    });
                    this.values.push(item.id);
                }

            this.$emit("change", this.values, items, this.selected);
            this.display = false;
        },
    },
};
</script>

<style></style>
