<template>
    <div v-if="opts">
        <v-select v-if="typeSelect == 'combobox'"
            ref="combobox"
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
            :hidden="disabled"
            :loading="comboItems.length==0"
        >
            <template v-slot:item="{ item, index }">
                <span v-if="opts.showID">{{ item.id }}. {{ item[opts.field] }}</span>
                <span v-else>{{ item[opts.field] }}</span>
            </template>
            <template v-slot:selection="{ item, index }">
                <v-chip :small="opts.dense" :close="!disabled" @click:close="removeItem($event,item,index)">
                    <span v-if="opts.showID">{{ item.id }}. {{ item[opts.field] }}</span>
                    <span v-else>{{ item[opts.field] }}</span>
                </v-chip>
            </template>
            <template v-slot:append v-if="opts['append-icon']">
                <v-icon @click.prevent="showTooltip($event, opts)">{{opts['append-icon']}}</v-icon>
                <v-icon>mdi-menu-down</v-icon>
            </template>
        </v-select>

        <v-autocomplete
            v-if="typeSelect == 'autocomplete' || typeSelect == 'tree' || typeSelect == 'auto'"
            ref="combobox"
            clearable
            auto-select-first
            @keydown="onKeyDown"
            @click:clear.prevent="onClear"
            :menu-props="{ bottom: true, offsetY: true }"
            :items="comboItems"
            v-model="values"
            :search-input.sync="autocompleteSearch"
            :multiple="opts.multiple"
            chips
            @change="changeCombobox"
            v-bind="opts"
            item-value="id"
            :item-text="opts.field"
            :rules="rules"
            :disabled="disabled"
        >
            <template v-slot:item="{ item, index }">
                <span v-if="opts.showID">{{ item.id }}. {{ item[opts.field] }}</span>
                <span v-else>{{ item[opts.field] }}</span>
            </template>
            <template v-slot:selection="{ item, index }">
                <v-chip :small="opts.dense" :close="!disabled" @click:close="removeItem($event,item,index)">
                    <span v-if="opts.showID">{{ item.id }}. {{ item[opts.field] }}</span>
                    <span v-else>{{ item[opts.field] }}</span>
                </v-chip>
            </template>
            <template v-slot:append>
                <v-btn class="mt-n1" v-if="typeSelect == 'tree'" @click.prevent="open()" fab x-small color="primary"><v-icon color="white">mdi-menu-down</v-icon></v-btn>
            </template>
        </v-autocomplete>

        <!-- TREE !-->
        <div v-if="typeSelect == 'tree2'" class="mb-4">
            <label>{{opts.label}}</label>
            <vue-treeselect v-model="values" :multiple="opts.multiple" :options="treeItems" :placeholder="opts.placeholder" :disabled="disabled">
            </vue-treeselect>
        </div>


        <v-select
            ref="combobox"
            v-if="typeSelect == 'table'"
            clearable
            @click:clear.prevent="onClear"
            @click.prevent="open()"
            :items="items"
            v-model="values"
            :multiple="true"
            chips
            item-value="value"
            item-text="text"
            v-bind="opts"
            :rules="rules"
            :disabled="disabled"
            :deletable-chips="true"
        >
            <template v-slot:item="{ item, index }">
                <span v-if="opts.showID">{{ item.value }}. {{ item.text }}</span>
                <span v-else>{{ item.text }}</span>
            </template>
            <template v-slot:selection="{ item, index }">
                <v-chip v-if="item.value || item.text" :small="opts.dense" :close="!disabled" @click:close="removeItem($event,item,index)">
                    <span v-if="opts.showID">{{ item.value }}. {{ item.text }}</span>
                    <span v-else>{{ item.text }}</span>
                </v-chip>
            </template>
            <template v-slot:append v-if="opts['append-icon']">
                <v-icon @click.prevent="showTooltip($event, opts)">{{opts['append-icon']}}</v-icon>
                <v-btn @click.prevent="open()" fab x-small color="primary"><v-icon color="white">mdi-menu-down</v-icon></v-btn>
            </template>
        </v-select>


        <v-dialog v-model="display" scrollable fullscreen>
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
                        <v-card-title class="pa-0 ma-0">
                            <v-toolbar dense color="primary" elevation="0">
                                <v-toolbar-title class="white--text">Выберите:</v-toolbar-title>
                                <v-spacer></v-spacer>
                                <v-btn icon @click="display = false"><v-icon color="white">close</v-icon></v-btn>
                            </v-toolbar>
                        </v-card-title>
                <v-card-text>
                <mdp-tree
                        ref="TableSelector"
                        v-model="values"
                        v-bind="opts.options || {}"
                        :tableCaption="opts.label"
                        :tableName="opts.table"
                        :fieldName="opts.field"
                        :fieldAltName="opts.field_alt"
                        :multiple="opts.multiple"
                        @select="rowSelected"
                >
                </mdp-tree>
                </v-card-text>
                         <v-divider />
                         <v-card-actions class="pa-4">
                            <v-btn class="float-right mr-4" color="red" @click="display = false">Закрыть</v-btn>
                            <v-spacer />
                            <v-btn
                                elevation="4"
                                color="primary"
                                @click="finishSelect()"
                                :disabled="selected.length == 0"
                                >Выбрать</v-btn>
                         </v-card-actions>
            </v-card>
            <!-- TREE !-->
        </v-dialog>
    </div>
</template>

<script>
import Vue from "vue";
import tableCache from "./tableCache.js";
import VueTreeselect from '@riophae/vue-treeselect'
import '@riophae/vue-treeselect/dist/vue-treeselect.css'

export default {
    components: {
        "mdp-table": () => import("../../table/Table"),
        "mdp-tree": () => import("./SelectFromTree.vue"),
        "vue-treeselect": VueTreeselect,
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
            if (JSON.stringify(this.options) == JSON.stringify(this.opts)) return;
            this.postRefresh();
        },
        value() {
            this.postRefresh();
        },
        items() {
            this.postRefresh();
        },
        autocompleteSearch(val) {
            if (this.typeSelect == "auto") return false;
            val && val !== this.values && this.autocompleteQuery(val);
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
            treeItems: [],
            autocompleteSearch: null,
            comboboxSelectedItems: [],
            values: [],
            labels: "",
            table_postfix: "",
        };
    },
    mounted() {
        this.postRefresh();
        setTimeout(()=>{
            if (this.opts && this.opts.table) tableCache.tables[this.opts.table+this.table_postfix] = null;
        }, 19000);
    },
    computed: {
    },
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
           this.$nextTick(()=>{ this.$refs.combobox.isMenuActive = false; });
        },

        postRefresh() {
            this.needRefresh = true;
            setTimeout(this.refresh, 200);
        },
        async refresh() {
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
                            value = value.replace(/\{\{(.*?)\}\}/gi, (match, name) => {
                                if (!this.row[name]) return 0;
                                return this.row[name];
                            });
                            e.value = value;
                        }
                    });
                }
            }

            this.values = this.value || [];
            if (typeof this.value != "object" && !isNaN(parseInt(this.value))) this.values = [parseInt(this.value)];

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
                rules = rules.replace(/\{\{(.*?)\}\}/gi, (match, name) => {
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

            if (this.typeSelect == "combobox" || this.typeSelect == "auto" || this.typeSelect == "tree2") {
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
                    this.opts.fieldraw.replace(/\{\{(.*?)\}\}/gi, (match, name) => {
                        this.opts.fields.push(name);
                    });
                    let filter = [];
                    if (this.opts.options && this.opts.options.tableFilter) filter = this.opts.options.tableFilter;
                    let fields = ["id", ...this.opts.fields];
                    if (this.opts.afterChange) fields = null;
                    this.$api("table", this.opts.table, "get", {fast:true, mini:true, fields:fields, limit:1000, filter }).then(response=>{
                        this.comboItems = response.rows.map(e=>{
                            e.text = this.opts.fieldraw.replace(/\[(.*?)\]/gi, (match, name) => e[name]!=undefined ? e[name] : "");
                            e.text = e.text.replace(/\{\{(.*?)\}\}/gi, (match, name) => e[name]!=undefined ? e[name] : "");
                            return e;
                        });
                        this.changeCombobox();
                    });
                } else {
                    let filter = [];
                    if (this.opts.options && this.opts.options.tableFilter) filter = this.opts.options.tableFilter;
                    this.table_postfix = filter.length>0 ? "__"+filter[0].field : "";

                    this.comboItems = [];
                    let fields = ["id", this.opts.field];
                    if (this.opts.afterChange) fields = null;

                    if (tableCache.tables[this.opts.table+this.table_postfix] && tableCache.tables[this.opts.table+this.table_postfix] === true) {
                        for (let i=0; i<9; i++) {
                            await new Promise((resolve, reject) => setTimeout(resolve,500) );
                            if (tableCache.tables[this.opts.table+this.table_postfix] !== true) break;
                        }
                    }
                    if (tableCache.tables[this.opts.table+this.table_postfix] && tableCache.tables[this.opts.table+this.table_postfix] !== true) {
                        this.comboItems = JSON.parse(JSON.stringify(tableCache.tables[this.opts.table+this.table_postfix]));
                    }
                    if (!tableCache.tables[this.opts.table+this.table_postfix]) {
                        tableCache.tables[this.opts.table+this.table_postfix] = true;
                        this.$api("table", (this.typeSelect == "tree2"?"tree/":"")+this.opts.table, "get", {fast:true, mini:true, fields:fields, limit:1000, filter }).then(response=>{
                            this.comboItems = response.rows;
                            if (this.typeSelect == "tree2") {
                              const treeMap = (e)=>{if (e.children && e.children.length>0) e.children = e.children.map(treeMap); e.label=e[this.opts.field]; return e;};
                              this.treeItems = response.rows.map(treeMap);
                            }
                            tableCache.tables[this.opts.table+this.table_postfix] = JSON.parse(JSON.stringify(this.comboItems));
                            this.changeCombobox();
                        });
                    }
                }
            }

            if (this.typeSelect == "autocomplete" || this.typeSelect == "tree") {
                let values = this.row[this.opts.name + "_values"];
                if (!values && this.row[this.opts.name + "_text"]) {
                    values = [{ value: this.row[this.opts.name], text: this.row[this.opts.name + "_text"] }];
                }
                
                if (this.comboItems.length==0 && this.values.length>0 && (!values || values.length == 0)) {
                    let fields = ["id", this.opts.field];
                    let filter = [{field:"id", oper:"in", value:this.values}];
                    this.$api("table", this.opts.table, "get", {fast:true, mini:true, fields:fields, limit:1000, filter }).then(response=>{
                        this.comboItems = response.rows;
                        this.comboboxSelectedItems = JSON.parse(JSON.stringify(this.comboItems));
                        this.changeCombobox();
                    });
                }
               
                if (values && values.length > 0) {
                    this.comboItems = [];
                    this.comboboxSelectedItems = [];
                    if (values)
                        values.forEach((e) => {
                            const el = {};
                            el.id = e.value;
                            el[this.opts.field] = e.text;
                            this.comboItems.push(el);
                            this.comboboxSelectedItems.push(el);
                        });
                }
                if (!this.opts.multiple && this.values && this.values.length>0) this.values = this.values[0];
            }

        },

        autocompleteQuery(searchValue) {
            if (searchValue.length == 0) return;
            let fields = ["id", this.opts.field];
            if (this.opts.afterChange) fields = null;
            this.comboItems = [];
            let filter = this.opts.options && this.opts.options.tableFilter ? this.opts.options.tableFilter : [];
            filter.push({ field: this.opts.field, oper: "like", value: searchValue });
            if (this.opts.onlyChilds) filter.push({ field: "parent_id", oper: ">", value: 0 });
            this.$api("table", this.opts.table, "get", {
                fast: true,
                fields: fields,
                filter,
                limit: 200,
            }).then((response) => {
                this.comboItems = [...this.comboboxSelectedItems, ...response.rows];
            });
        },

        onKeyDown(evt) {
            if (this.opts.multiple && evt.keyCode == 32) {
               evt.preventDefault();
               let selNdx = this.$refs.combobox.$refs.menu.listIndex;
               this.$refs.combobox.setMenuIndex(selNdx);
               this.$refs.combobox.selectItem(this.$refs.combobox.items[selNdx]);
            }
            if (this.opts.multiple && evt.keyCode == 13) {
               this.$nextTick(()=>{ 
                   this.$refs.combobox.isMenuActive = false; 
                   let selNdx = this.$refs.combobox.$refs.menu.listIndex;
                   this.$refs.combobox.setMenuIndex(selNdx);
                   let selID = this.$refs.combobox.items[selNdx].id;
                   this.$nextTick(()=>{ 
                       if (Array.isArray(this.values) && !this.values.includes(selID)) this.$refs.combobox.selectItem(this.$refs.combobox.items[selNdx]);
                   });
               });
            }
        },

        changeCombobox() {
            if (this.opts.multiple) {
                this.comboboxSelectedItems = this.comboItems.filter((e) => this.values.includes(e.id));
            } else {
                this.comboboxSelectedItems = this.comboItems.filter((e) => this.values == e.id);
            }

            let items = [];
            for (let item of this.comboboxSelectedItems) {
                    items.push({
                        value: item.id,
                        text: item[this.opts.field],
                    });
            }
            this.$emit("change", this.values, items, this.comboboxSelectedItems);
        },

        onClear() {
            this.$nextTick().then(() => {
                this.$emit("change", [], []);
                this.display = false;
            });
        },
        removeItem(evt, item, index) {
            if (isNaN(parseInt(index))) return;
            if (!this.opts.multiple) { 
                this.values = null; 
                this.changeCombobox(); 
                return; 
            }
            this.values.splice(index, 1);
            this.changeCombobox();
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
                        text = this.opts.field.replace(/\[(.*?)\]/gi, (match, name) => item[name] );
                        text = text.replace(/\{\{(.*?)\}\}/gi, (match, name) => item[name]);
                    }
                    items.push({
                        value: item.id,
                        text: text,
                    });
                    this.values.push(item.id);
                }

            this.comboboxSelectedItems = this.selected;

            if (this.opts.multiple) {
                this.$emit("change", this.values, items, this.selected);
            } else {
                this.values = this.values[0];
                this.$emit("change", this.values, items, this.selected);
            }
            this.display = false;
        },
    },
};
</script>

<style></style>
