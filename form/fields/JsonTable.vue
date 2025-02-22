<template>
    <div id="json-table" class="mb-4">
        <v-card>
            <v-sheet class="py-2 pl-4 pr-1 body-1" color="primary white--text">
                <v-card-title class="pa-0">
                    {{ options.label }}
                    <v-spacer />
                    <v-btn v-if="options.multiple && !options.json.readonly" fab small color="green" @click="addNewRow"><v-icon>add</v-icon></v-btn>
                </v-card-title>
            </v-sheet>

            <!-- multi line table !-->
            <table v-if="options.multiple">

<draggable ghost-class="ghost" v-model="dataList" group="*" @start="drag = true" @end="drag = false" handle=".handle" @change="updateSort()">
                <tr v-for="(dataRow,ndx) in dataList" :key="ndx">
                    <td class="handle"><v-icon>mdi-menu</v-icon>
                        <div class="d-flex">
                        <v-icon small @click="itemUp(ndx)">mdi-arrow-up</v-icon>
                        <v-icon small @click="itemDown(ndx)">mdi-arrow-down</v-icon>
                        </div>
                    </td>
                    <td v-for="(col,i) in options.json.columns" :key="i" :width="col.width">
                        <mdp-form-field v-if="col.type" v-model="dataList[ndx][col.name]" :row="dataList[ndx]" :options="col" :name="col.name" @change="change" @changeField="changeField($event,ndx,col)" :disabled="options.json.readonly || col.disabled" />
                        <textarea v-else rows="1" v-model="dataList[ndx][col.name]" :placeholder="col.placeholder || col.label" @input="change" :disabled="options.json.readonly || col.disabled" />
                        <div v-if="col.buttons" class="d-flex">
                            <div v-for="(btn,k) in col.buttons" :key="k" class="click-btns" @click="addText(ndx, col, btn, $event)">{{btn}}</div>
                        </div>
                    </td>
                    <td>
                        <v-btn v-if="!options.json.readonly" fab x-small color="red" @click="deleteRow(ndx)"><v-icon>delete</v-icon></v-btn>
                    </td>
                </tr>
                <tr slot="header"> 
                    <th style="width:1px"></th>
                    <th v-for="(item,i) in options.json.columns" :key="i">{{item.header || item.label || ''}}</th>
                    <th style="width:1px"></th>
                </tr>
</draggable> 

            </table>

            <!-- single line table short !-->
            <table v-if="!options.multiple && dataList.length > 0 && options.json.columns.length <= 8" class="">
                <tr>
                    <td v-for="(col,i) in options.json.columns" :key="i" :width="col.width">
                        <label>{{col.header || col.label || ''}}</label>
                    </td>
                </tr>
                <tr>
                    <td v-for="(col,i) in options.json.columns" :key="i">
                        <mdp-form-field v-if="col.type" v-model="dataList[0][col.name]" :row="dataList[0]" :options="col" :name="col.name" @change="change" :disabled="options.json.readonly || col.disabled" />
                        <textarea v-else rows="1" v-model="dataList[0][col.name]" :placeholder="col.placeholder || col.label" @input="change" :disabled="options.json.readonly || col.disabled" />
                    </td>
                </tr>
            </table>

            <!-- single line table long !-->
            <div v-if="!options.multiple && dataList.length > 0 && options.json.columns.length > 8" class="d-flex flex-wrap">
                    <div v-for="(col,i) in options.json.columns" :key="i" class="d-flex flex-wrap ml-2 mr-1 mb-1" style="width:140px">
                        <label>{{col.header || col.label || ''}}</label>
                        <mdp-form-field v-if="col.type" v-model="dataList[0][col.name]" :row="dataList[0]" :options="col" :name="col.name" @change="change" :disabled="options.json.readonly || col.disabled" />
                        <textarea v-else rows="1" v-model="dataList[0][col.name]" :placeholder="col.placeholder || col.label" @input="change" :disabled="options.json.readonly || col.disabled" />
                    </div>
            </div>

        </v-card>
    </div>
</template>

<script>
import draggable from "vuedraggable";
export default {
    components: {
        "mdp-form-field":() => import("../FormField.vue"),
        draggable,
    },
    props: {
        value: { required: true },
        options: { type: Object, default: null },
        row: { required: true },
        rules: { type: Array, default: null },
        disabled: { default: false },
    },
    data() {
        return {
            dataList: [],
            localRow: [],
        };
    },
    watch: {
        value() {
            this.refresh();
        },
    },
    mounted() {
        this.refresh();
        if (this.rules && this.$parent.$parent.register) this.$parent.$parent.register(this);
        setTimeout(()=>{
            const elems = document.querySelectorAll("#json-table textarea");
            elems.forEach((el)=>{ this.resizeTextArea(el); });
        },100);
    },
    beforeDestroy() {
        if (this.rules && this.$parent.$parent.register) this.$parent.$parent.unregister(this);
    },
    
    methods: {
        refresh() {
            if (typeof this.value == "string") {
                try {
                    this.dataList = JSON.parse(this.value);
                } catch (e) {
                    this.dataList = [];
                }
            } else {
                this.dataList = this.value || [];
            }

            if (this.options.json && this.options.json.rows && this.options.json.rows.length > 0 && this.dataList.length == 0) {
                this.options.json.rows.forEach((e) => {
                    this.addNewRow(e);
                });
            }

            if (!this.options.multiple && this.dataList.length == 0) {
                this.addNewRow();
            }

            this.change();
        },

        addNewRow(data) {
            const line = {};
            if (!this.options.json) return;
            this.options.json.columns.forEach((e) => {
                line[e.name] = data && data[e.name] ? data[e.name] : "";
            });
            this.dataList.push(line);
        },

        deleteRow(index) {
            this.dataList = this.dataList.filter((obj, ndx) => {
                return ndx !== index;
            });
            this.change();
        },

        addText(ndx, col, btn, event){
            this.dataList[ndx][col.name] = this.dataList[ndx][col.name] + btn + " ";
            this.change();
        },

        change(evt, val, txt){
            this.$emit("input", this.dataList);
            if (evt) this.$nextTick(this.resizeTextArea(evt.target));
        },
        changeField(evt, ndx, col){
            this.$emit("fieldChange", evt, ndx, col);
        },

        resizeTextArea(el){
            if (!el) return;
            el.style.cssText = "height:auto;";
            el.style.cssText = "height:" + el.scrollHeight + "px;";
        },


        validate() {
            if (this.disabled) return true;
            if (this.rules) {
                for (let index = 0; index < this.rules.length; index++) {
                    const rule = this.rules[index];
                    const valid = typeof rule === 'function' ? rule(this.dataList) : rule;
          
                    if (valid === false || typeof valid === 'string') {
                        return false
                    }
                }
            }
            return true;
        },
        updateSort() {
            this.$emit("input", this.dataList);
        },
        itemUp(ndx) {
            if (ndx==0) return;
            let item1 = JSON.parse(JSON.stringify(this.dataList[ndx]));
            let item2 = JSON.parse(JSON.stringify(this.dataList[ndx-1]));
            this.dataList[ndx-1] = item1;
            this.dataList[ndx] = item2;
            this.$forceUpdate();
            this.updateSort();
        },
        itemDown(ndx) {
            if (ndx==this.dataList.length-1) return;
            let item1 = JSON.parse(JSON.stringify(this.dataList[ndx]));
            let item2 = JSON.parse(JSON.stringify(this.dataList[ndx+1]));
            this.dataList[ndx+1] = item1;
            this.dataList[ndx] = item2;
            this.$forceUpdate();
            this.updateSort();
        },

    },
};
</script>

<style scoped>
#json-table table {
    width: 100%;
    border-radius: 3px;
    border-collapse: collapse;
    border: 1px solid #d0d0d0;
}
#json-table table th {
    text-align: center;
    font-size: 16px;
    background: var(--v-background-base);
    padding: 10px;
    border-left: 1px solid #d0d0d0;
}
#json-table table td {
    vertical-align: top;
    text-align: left;
    padding: 5px;
    border-left: 1px solid #d0d0d0;
}
#json-table table tr {
    border-bottom: 1px solid #d0d0d0;
}
#json-table table tr:hover {
    background: var(--v-accent-lighten4);
}

#json-table table th,
tr:not(:last-child) {
}

#json-table  label { padding-left:2px; }
#json-table  textarea { 
    width:100%; padding:4px; 
    overflow:hidden;
    border-width:1px; border-style:solid; border-color: var(--v-primary-base); border-radius:4px;  
}

#json-table .click-btns { background:var(--v-background-darken2); margin:2px 1px 0 3px; padding:1px 5px; border-radius:4px; cursor:pointer; 
    height: 18px;
    font-size: 12px;
    line-height: 16px;
}
.handle {
    cursor: pointer;
}
</style>
