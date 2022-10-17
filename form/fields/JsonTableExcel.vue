<template>
    <div class="json-table mb-6">
        <v-card v-if="options">
            <v-sheet class="py-1 pl-4 pr-1 body-1" color="primary white--text">
                <v-card-title class="pa-0">
                    {{ options.label }}
                    <v-spacer />
                    <v-btn v-if="options.multiple && !options.json.readonly" fab x-small color="green" @click="addNewRow"><v-icon>add</v-icon></v-btn>
                </v-card-title>
            </v-sheet>
            <div :id="'json-excel-table-'+options.name" ref="table"></div>
        </v-card>
    </div>
</template>

<script>
import {TabulatorFull as Tabulator} from 'tabulator-tables';
import "tabulator-tables/dist/css/tabulator.css";

export default {
    props: {
        value: { required: true },
        options: { type: Object, default: null },
        row: { required: true },
        disabled: { default: false },
    },
    data() {
        return {
            table: null,
            rows: [],
        };
    },
    computed: {
        columnsArray() {
            return Object.keys(this.options.json.columns).map((key) => {
                let type = "string";
                if (this.options.json.columns[key].type == "dateTime") type = "datetime";
                if (this.options.json.columns[key].type == "linkTable") type = "map";
                if (this.options.json.columns[key].type == "select") type = "map";
                if (this.options.json.columns[key].type == "integer") type = "number";
                if (this.options.json.columns[key].type == "float") type = "number";
                this.options.json.columns[key].type = type;
                return this.options.json.columns[key];
            });
        },
    },
    watch: {
        value() {
            this.refresh();
        },
    },
    mounted() {
        this.refresh();
    },

    methods: {
        async refresh() {
            if (typeof this.value == "string") {
                try {
                    this.rows = JSON.parse(this.value);
                } catch (e) {
                    this.rows = [];
                }
            } else {
                this.rows = this.value || [];
            }

            if (this.options.json.rows && this.options.json.rows.length > 0 && this.rows.length == 0) {
                this.options.json.rows.forEach((e) => {
                    this.addNewRow(e);
                });
            }

            if (this.rows.length == 0) {
                this.addNewRow();
            }

            let columns = await Promise.all(
                    Object.keys(this.options.json.columns).map(async (fldName) => {
                        const fldParam = { field: this.options.json.columns[fldName].name, title: this.options.json.columns[fldName].label };
                        fldParam.editor = true;
                        fldParam.sorter = "string";
                        fldParam.width = this.options.json.columns[fldName].width || 150;
                        fldParam.visible = this.options.json.columns[fldName].hidden ? false : true;
                        fldParam.headerFilter = false;

                        if (this.options.json.columns[fldName].type == "integer" || this.options.json.columns[fldName].type == "double" || this.options.json.columns[fldName].type == "float") {
                            fldParam.sorter = "number";
                            fldParam.editor = "number";
                            fldParam.mutator = (value, data)=>{ let val = parseFloat(value); return isNaN(val) ? 0 : val; }
                        }

                        if (this.options.json.columns[fldName].type == "select") {
                            fldParam.editor = "list";
                            fldParam.editorParams = {values:this.options.json.columns[fldName].items, clearable:true };
                            fldParam.formatter=(cell,params)=>{
                                return this.options.json.columns[fldName].items[cell.getValue()];
                            };
                        }
                        if (this.options.json.columns[fldName].type == "linkTable") {
                            this.options.json.columns[fldName].items = {};
                            let responseData = await this.$api("table", this.options.json.columns[fldName].table, "get", { fast: true, mimi: true });
                            responseData.rows.forEach((respRow) => {
                                this.options.json.columns[fldName].items[respRow.id] = respRow[this.options.json.columns[fldName].field];
                            });

                            fldParam.editor = "list";
                            fldParam.editorParams = {values:this.options.json.columns[fldName].items, clearable:true };
                            fldParam.formatter=(cell,params)=>{
                                return this.options.json.columns[fldName].items[cell.getValue()];
                            };
                            fldParam.formatter=(cell,params)=>{
                                let fldName = cell._cell.column.field;
                                let row = cell._cell.row.data;
                                return row[fldName+"_text"];
                            };
                        }

                        return fldParam;
                    })
            );

            if (this.options.json.group) {
                columns = this.options.json.group.map(grp=>{
                    grp.cols = columns.filter(e=>grp.columns.includes(e.field)); 
                    return {title:grp.label, columns:grp.cols};
                });
            }

            this.table = new Tabulator("#json-excel-table-"+this.options.name, {
                reactiveData: false,
             	layout: "fitColumns",
             	height: 90,
             	data: this.rows,
             	columns
            });

            this.table.on("cellEdited", (cell)=>{ 
           	    this.change();
            });
        },

        addNewRow(data) {
            const line = {};
            if (!this.options.json) return;
            this.options.json.columns.forEach((e) => {
                line[e.name] = data && data[e.name] ? data[e.name] : "";
            });
            this.rows.push(line);
        },

        change(evt) {
            //this.$emit("input", JSON.stringify(this.rows));
            this.$emit("input", this.rows);
        },
    },
};
</script>

<style>
.tabulator {
    height: auto  !important;
}

.tabulator-row:hover{
    background: rgba(0,32,51,0.08) !important;

}

.tabulator-row.tabulator-row-even {
    background-color: #E3F2FD;
}

.tabulator-col-title-holder {text-align:center;}
</style>