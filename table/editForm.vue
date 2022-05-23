<template>
    <v-card v-if="visible">
        <v-card-text style="padding:20px 20px 10px 20px;">
            <v-form ref="form" v-model="form_valid" class="d-flex flex-wrap">
                <template v-for="(item, i) in columnsFiltered">
                    <div style="width:100%" v-if="item.divider && item.vifResult">
                        <v-divider class="mt-5"></v-divider>
                        <p class="text-center title mb-4">{{ item.divider }}</p>
                    </div>
                    <mdp-form-field
                        :options="item"
                        :key="i"
                        :name="item.name"
                        v-model="localRow[item.name]"
                        :row="localRow"
                        :disabled="item.protected || item.disabled || action=='view'"
                        @change="fieldChange"
                        @changeRowField="changeRowField"
                        @changeFieldOption="changeFieldOption"
                    />
                </template>
            </v-form>
        </v-card-text>

        <v-overlay :absolute="true" :value="isLoading"> </v-overlay>
    </v-card>
</template>

<script>
import { mapActions } from "vuex";

export default {
    components: {
        "mdp-form-field": () => import("mapdap/form/FormField.vue"),
    },

    props: {
        visible: { type: Boolean, default: false },
        action: { type: String, default: "add" },
        tableName: { type: String, default: "" },
        tableInfo: { type: Object, default: null },
        rowId: { type: Number, default: 0 },
        row: { type: Object, default: null },
        page: { type: Number, default: 1 },
        buttons: {
            type: Object,
            default: () => {
                return {
                    left: { text: "Отмена", color: "red", icon: "close" },
                    right: { text: "Сохранить", color: "primary", icon: "save" },
                };
            },
        },
    },

    watch: {
        tableName() {
            this.loadRow();
        },
        tableInfo() {
            this.loadRow();
        },
        rowId() {
            this.loadRow();
        },
        visible() {
            this.loadRow();
        },
    },

    data() {
        return {
            pagination: false,
            form_valid: true,
            can_save: false,
            columns: [],
            isLoading: false,
            localRow: {},
        };
    },

    computed:{
        columnsFiltered(){
            if (!this.pagination) return this.columns;
            return this.columns.filter(e=>{ 
               if (!e.page) return true;
               return e.page == this.page;
            });
        },
    },

    mounted() {
        this.loadRow();
        document.addEventListener('keyup', this.onKeyUp);
    },
    beforeDestroy() {
        document.removeEventListener('keyup', this.onKeyUp);
    },

    methods: {
        loadRow() {
            this.can_save = false;
            if (this.action == "delete") this.can_save = true;

            this.localRow = this.row || {};
            this.columns = [];
            this.columns = this.convertColumns(this.tableInfo.columns);
            this.pagination = this.tableInfo.pagination || 0;
            this.pagination = parseInt(this.pagination) > 1 ? parseInt(this.pagination) : 0;
            this.$emit("setpagination", this.pagination);
            this.localRow = JSON.parse(JSON.stringify(this.localRow));
            if (this.rowId == 0) return;

            this.isLoading = true;
            this.$api("table", this.tableName, this.rowId + "/?mini=true")
                .then((response) => {
                    this.isLoading = false;
                    if (response.rows[0]) this.localRow = response.rows[0];

                    this.$nextTick().then(() => {
                        this.$forceUpdate();
                    });
                })
                .catch((error) => {
                    this.isLoading = false;
                });
        },

        convertColumns(columns) {
            var rez = [];
            for (let item in columns) {
                var obj = columns[item];
                if (obj.protected && !obj.visible) {
                    continue;
                }
                obj.vifResult = true;
                obj.name = item;
                rez.push(obj);
                if (!this.localRow[item]) this.localRow[item] = null;
            }
            return rez;
        },

        fieldChange(name, value, text) {
            this.can_save = true;
            this.$emit("change");
        },

        changeRowField(name, value, text){
            this.localRow[name] = value;
            if (text) this.localRow[name+'_text'] = text;
            this.$forceUpdate();
        },

        changeFieldOption(fld, opt, val) {
            this.columns.forEach((e) => {
                if (e.name == fld) e[opt] = val;
            });
            this.$forceUpdate();
        },

        onKeyUp(event){
            //console.log(event)
            if (event.key=="Escape") {
                if (!this.can_save) this.$emit("close");
            }
            if (event.ctrlKey && event.key=="Enter" && this.can_save) {
                this.save();
            }
        },

        isFormValid() {
            return this.$refs.form.validate();
        },

        save() {
            if (!this.isFormValid()) {
                this.$swal.toastError("Заполните все поля правильно", "center-center", 1500);
                return;
            }

            this.$emit("save", this.localRow);
        },
    },
};
</script>
