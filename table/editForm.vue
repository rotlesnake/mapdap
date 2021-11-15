<template>
    <v-card v-if="visible">
        <v-card-text style="padding:20px 20px 10px 20px;">
            <v-form ref="form" v-model="form_valid">
                <template v-for="(item, i) in columns">
                    <mdp-form-field
                        :options="item"
                        :key="i"
                        :name="item.name"
                        v-model="row[item.name]"
                        :row="row"
                        :disabled="item.protected || item.disabled"
                        @change="fieldChange"
                    />
                </template>
            </v-form>
        </v-card-text>

        <v-divider></v-divider>
        <v-card-actions>
            <v-btn class="mx-2" :color="buttons.left.color" @click="$emit('close')">
                <v-icon left>{{ buttons.left.icon }}</v-icon> {{ buttons.left.text }}
            </v-btn>
            <v-spacer></v-spacer>
            <v-btn class="mx-2" :color="buttons.right.color" :disabled="!can_save" @click="save">
                <v-icon left>{{ buttons.right.icon }}</v-icon> {{ buttons.right.text }}
            </v-btn>
        </v-card-actions>

        <v-overlay :absolute="true" :value="isLoading"> </v-overlay>
    </v-card>
</template>

<script>
import { mapActions } from "vuex";

export default {
    components: {
        "mdp-form-field": () => import("../form/FormField.vue"),
    },

    props: {
        visible: { type: Boolean, default: false },
        action: { type: String, default: "add" },
        tableName: { type: String, default: "" },
        tableInfo: { type: Object, default: null },
        rowId: { type: Number, default: 0 },
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
            form_valid: true,
            can_save: false,
            columns: [],
            isLoading: false,
            row: {},
        };
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

            this.row = {};
            this.columns = [];
            this.columns = this.convertColumns(this.tableInfo.columns);
            this.row = JSON.parse(JSON.stringify(this.row));
            if (this.rowId == 0) return;

            this.isLoading = true;
            this.$api("table", this.tableName, this.rowId + "/?mini=true")
                .then((response) => {
                    this.isLoading = false;
                    if (response.rows[0]) this.row = response.rows[0];
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
                obj.name = item;
                rez.push(obj);
                if (!this.row[item]) this.row[item] = null;
            }
            return rez;
        },

        fieldChange(name, value, text) {
            this.can_save = true;
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

        save() {
            this.$refs.form.validate();
            if (!this.form_valid) {
                this.$swal.toastError("Заполните все поля правильно", "center-center", 1500);
                return;
            }

            this.$emit("save", this.row);
        },
    },
};
</script>
