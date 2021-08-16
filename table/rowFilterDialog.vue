<template>
    <v-dialog v-model="visible" scrollable persistent max-width="990px">
        <v-card v-if="visible">
            <v-toolbar dense dark color="primary" elevation="0">
                <v-toolbar-title>Фильтр данных</v-toolbar-title>
                <v-spacer></v-spacer>
                <v-btn icon dark @click="$emit('close')">
                    <v-icon>close</v-icon>
                </v-btn>
            </v-toolbar>

            <v-card-text style="padding:5px 20px 10px 20px;">
                <v-form>
                    <template v-for="(item, i) in columns">
                        <mdp-form-field
                            :options="item"
                            :key="i"
                            :name="item.name"
                            v-model="row[item.name]"
                            :row="row"
                            @change="fieldChange"
                        />
                    </template>
                </v-form>
            </v-card-text>

            <v-divider></v-divider>
            <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn class="mx-2" color="primary" @click="save"> <v-icon left>close</v-icon> Применить </v-btn>
                <v-spacer></v-spacer>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script>
import { mapActions } from "vuex";

export default {
    components: {
        "mdp-form-field": () => import("../form/FormField.vue"),
    },

    props: {
        visible: { type: Boolean, default: false },
        value: { type: Array, default: null },
        info: { type: Object, default: null },
        row: { type: Object, default: null },
    },

    data() {
        return {
            columns: [],
        };
    },

    watch: {
        info() {
            this.refresh();
        },
        value() {
            this.refresh();
        },
        row() {
            this.refresh();
        },
    },

    mounted() {
        this.refresh();
    },

    methods: {
        refresh() {
            this.columns = [];
            if (!this.info) return;

            for (let i in this.info.filter) {
                let name = i;
                if (this.info.filter[i].name) name = this.info.filter[i].name;

                let col = JSON.parse(JSON.stringify(this.info.columns[name]));
                col = { ...col, ...this.info.filter[i] };
                if (col.filterType == "in") col.multiple = true;
                col.protected = false;
                this.columns.push(col);
            }
        },

        fieldChange(name, value, text) {
            //console.log(name, value, text);
            //console.log(this.row);
        },

        save() {
            let result = [];
            this.columns.forEach((e) => {
                if (!this.row[e.name]) return;
                if (this.row[e.name].length == 0) return;
                result.push({ field: e.name, oper: e.filterType, value: this.row[e.name] });
            });
            this.$emit("input", result);
            this.$emit("change", this.row);
        },
    },
};
</script>

<style>
.as-disabled label {
    color: #909090 !important;
    text-decoration: line-through !important;
}
.as-disabled .v-icon {
    color: #c5c5c5 !important;
}
.as-enabled .v-icon {
    color: #909090 !important;
}
</style>
