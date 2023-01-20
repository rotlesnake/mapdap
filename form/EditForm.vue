<template>
    <v-card>
        <v-card-text style="padding: 20px 20px 10px 20px">
            <v-form ref="form" v-model="form_valid" class="d-flex flex-wrap">
                <template v-for="(item, i) in localColumns">
                    <div style="width: 100%" v-if="item.divider && item.vifResult">
                        <v-divider class="mt-5"></v-divider>
                        <p class="text-center title mb-4">{{ item.divider }}</p>
                    </div>
                    <mdp-form-field
                        :options="item"
                        :key="i"
                        :name="item.name"
                        v-model="localRow[item.name]"
                        :row="localRow"
                        :disabled="item.protected || item.disabled || action == 'view'"
                        @change="fieldChange"
                        @changeRowField="changeRowField"
                        @changeFieldOption="changeFieldOption"
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
        columns: { type: Array, default: null },
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
        row() {
            this.refresh();
        },
        columns() {
            this.refresh();
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

            this.localRow = JSON.parse(JSON.stringify(this.row));
            if (this.columns instanceof Array) {
                this.localColumns = JSON.parse(JSON.stringify(this.columns));
            } else {
                if (this.columns) this.localColumns = JSON.parse(JSON.stringify( Object.values(this.columns) ));
            }
        },


        fieldChange(name, value, text) {
            this.can_save = true;
            this.$emit("change");
        },
        changeRowField(name, value, text) {
            this.localRow[name] = value;
            if (text) this.localRow[name + "_text"] = text;
            this.$forceUpdate();
        },
        changeFieldOption(fld, opt, val) {
            this.localColumns.forEach((e) => {
                if (e.name == fld) e[opt] = val;
            });
            this.$forceUpdate();
        },

        save() {
            this.$refs.form.validate();
            if (!this.form_valid) {
                this.$swal.toastError("Заполните все поля правильно", "center-center", 1500);
                return;
            }
            this.$emit("save", this.localRow);
        },
    },
};
</script>
