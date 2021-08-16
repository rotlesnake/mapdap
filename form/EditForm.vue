<template>
    <v-card>
        <v-card-text style="padding:20px 20px 10px 20px;">
            <v-form ref="form" v-model="form_valid">
                <template v-for="(item, i) in columns">
                    <mdp-form-field
                        :options="item"
                        :key="i"
                        :name="item.name"
                        v-model="row[item.name]"
                        :row="row"
                        :disabled="item.protected"
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
        "mdp-form-field": () => import("./FormField.vue"),
    },

    props: {
        action: { type: String, default: "add" },
        columns: { type: Object, default: null },
        row: { type: Object, default: null },
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
        columns() {
            this.loadRow();
        },
    },

    data() {
        return {
            form_valid: true,
            can_save: false,
            isLoading: false,
            localRow:{},
            localColumns:[],
        };
    },

    mounted() {
        this.refresh();
    },

    methods: {
        refresh() {
            this.can_save = false;
            if (this.action == "delete") this.can_save = true;

            localRow = JSON.parse(JSON.stringify(this.row));
            localColumns = JSON.parse(JSON.stringify(this.columns));
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
