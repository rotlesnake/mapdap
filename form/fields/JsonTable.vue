<template>
    <div id="json-table" class="mb-4">
        <v-card>
            <v-sheet class="py-2 pl-4 pr-1 body-1" color="primary white--text">
                <v-card-title class="pa-0">
                    {{ options.label }}
                    <v-spacer />
                    <v-btn v-if="options.multiple" fab small color="green" @click="addNewRow"><v-icon>add</v-icon></v-btn>
                </v-card-title>
            </v-sheet>

            <table v-if="options.multiple">
                <tr>
                    <th v-for="(item,i) in options.json.columns" :key="i">{{item.label}}</th>
                    <th style="width:1px"></th>
                </tr>

                <tr v-for="(dataRow,ndx) in dataList" :key="ndx">
                    <td v-for="(col,i) in options.json.columns" :key="i">
                        <input
                            v-model="dataList[ndx][col.name]"
                            @input="change"
                        />
                    </td>
                    <td>
                        <v-btn fab x-small color="red" @click="deleteRow(ndx)"><v-icon>delete</v-icon></v-btn>
                    </td>
                </tr>
            </table>

            <div v-else class="d-flex flex-wrap">
                <div v-for="(dataRow,ndx) in dataList" :key="ndx" class="d-flex flex-wrap" >
                    <div v-for="(col,i) in options.json.columns" :key="i" class="d-flex flex-wrap ml-2 mr-1 mb-1" style="width:140px">
                        <label>{{col.label}}</label>
                        <input
                            v-model="dataList[ndx][col.name]"
                            :placeholder="col.label"
                            @input="change"
                        />
                    </div>
                </div>
            </div>

        </v-card>
    </div>
</template>

<script>
export default {
    props: {
        value: { required: true },
        options: { type: Object, default: null },
        row: { required: true },
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
            if (!this.options.multiple && this.dataList.length == 0) {
                this.addNewRow();
            }
        },

        addNewRow() {
            const line = {};
            this.options.json.columns.forEach(e=>{
                line[e.name] = "";
            });
            this.dataList.push(line);
        },

        deleteRow(index) {
            this.dataList = this.dataList.filter((obj, ndx) => {
                return ndx !== index;
            });
            this.change();
        },
        change(){
            this.$emit("input", JSON.stringify(this.dataList));
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
#json-table  input { 
    width:100%; padding:4px; 
    border-width:1px; border-style:solid; border-color: var(--v-primary-base); border-radius:4px;  
}

</style>
