<template>
    <div v-if="opts">
        <v-select v-if="typeSelect == 'combobox'"
            clearable
            @click:clear.prevent="onClear"
            :items="comboItems"
            v-model="values"
            :multiple="opts.multiple"
            chips
            @change="changeCombobox"
            v-bind="opts"
            item-value="id"
            :item-text="opts.field"
            :rules="rules"
            :disabled="disabled"
            :loading="comboItems.length==0"
        >
            <template v-slot:selection="{ item, index }">
                <v-chip :small="opts.dense">
                    <span>{{ item[opts.field] }}</span>
                </v-chip>
            </template>
            <template v-slot:append v-if="opts['append-icon']">
                <v-icon @click.prevent="showTooltip($event, opts)">{{opts['append-icon']}}</v-icon>
                <v-icon>mdi-menu-down</v-icon>
            </template>
        </v-select>
        <v-select
            v-else
            @mouseup="open()"
            clearable
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
            <template v-slot:append v-if="opts['append-icon']">
                <v-icon @click.prevent="showTooltip($event, opts)">{{opts['append-icon']}}</v-icon>
                <v-icon>mdi-menu-down</v-icon>
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
        clickAppend: { type: Function, default: null },
    },
    watch: {
        row:{
            immediate: true,
            deep: true,
            handler(newVal, oldVal) { this.postRefresh(); }
        },
        options() {
            this.postRefresh();
        },
        value() {
            this.postRefresh();
        },
        items() {
            this.postRefresh();
        },
    },
    data() {
        return {
            needRefresh: false,
            typeSelect: 'table',
            display: false,
            selected: [],
            opts: null,
            comboItems: [],
            values: [],
            labels: "",
        };
    },
    mounted() {
        this.postRefresh();
    },
    computed: {},
    methods: {
        trimStr(str) {
            str = str.replace(/^\s+|\s+$/g,"");
            if (str.indexOf('[') > -1) {
                str = str.replace(/\[/gi, "").replace(/\]/gi, "");
                return str.split(",");
            }
            return str;
        },
        showTooltip(evt, options){
            if (this.clickAppend) this.clickAppend(evt, options);
            this.$nextTick(()=>{ this.$refs.combobox.isMenuActive = false; this.display = false; });
        },
        open(){ 
           if (this.disabled) return;
           this.display = true;
        },
        postRefresh() {
            this.needRefresh = true;
            this.$nextTick(this.refresh);
        },
        refresh() {
            if (!this.needRefresh) return;
            this.needRefresh = false;
            if (this.opts && this.opts.table != this.options.table) {
                this.comboItems = [];
            }
            this.opts = JSON.parse(JSON.stringify(this.options));

            this.typeSelect = "table";
            if (this.opts.typeSelect) this.typeSelect = this.opts.typeSelect;

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

            this.values = this.value || [];
            if (typeof this.value != "object") this.values = [parseInt(this.value)];

            if (this.opts.tableFilter && this.opts.tableFilter.length>2) {
                if (!this.opts.options) this.opts.options = {};
                this.opts.options.tableFilter = [];
                let rules = this.opts.tableFilter;
                rules = rules.replace(/\[(.*?)\]/gi, (match, name) => {
                    if (!this.row[name]) return 0;
                    if (typeof this.row[name] == "object" && this.row[name].length==0) return 0;
                    if (typeof this.row[name] == "object" && rules.indexOf("'in'")>-1 ) return "["+this.row[name]+"]";
                    return this.row[name];
                });
                var thisContext = this;
                rules = rules.split(" ;; ");
                rules.forEach(e=>{
                    let str = e.replace(/\(/gi, "").replace(/\)/gi, "").replace(/\'/gi, "");
                    let arr = str.split(",");
                    this.opts.options.tableFilter.push({field:this.trimStr(arr[0]), oper:this.trimStr(arr[1]), value:this.trimStr(arr[2])});
                });
            }

            if (this.typeSelect == "combobox") {
                if (!this.values || this.values.length==0) this.values = 0;
                if (!this.opts.multiple && this.values && this.values.length>0) this.values = this.values[0];
                if (this.opts.field.indexOf('[') > -1) {
                    this.opts.fieldraw = this.opts.field;
                    this.opts.field = "text";
                }
                if (this.comboItems.length > 0) return;
                if (this.opts.fieldraw && this.opts.fieldraw.indexOf('[') > -1) {
                    this.comboItems = [];
                    this.opts.fields = [];
                    this.opts.fieldraw.replace(/\[(.*?)\]/gi, (match, name) => {
                        this.opts.fields.push(name);
                    });
                    this.$api("table", this.opts.table, "get", {fast:true, fields:["id", ...this.opts.fields] }).then(response=>{
                        this.comboItems = response.rows.map(e=>{
                            e.text = this.opts.fieldraw.replace(/\[(.*?)\]/gi, (match, name) => e[name]);
                            return e;
                        });
                    });
                } else {
                    this.comboItems = [];
                    this.$api("table", this.opts.table, "get", {fast:true, fields:["id", this.opts.field] }).then(response=>{
                        this.comboItems = response.rows;
                    });
                }
            }
        },
        changeCombobox() {
            this.$emit("change", this.values, []);
        },
        onClear() {
            this.$nextTick().then(() => {
                this.$emit("change", [], []);
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
