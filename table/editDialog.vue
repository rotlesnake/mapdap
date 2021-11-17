<template>
    <v-dialog v-model="visible" scrollable persistent max-width="990px">
        <v-card v-if="visible">
            <v-card-title class="pa-0">
                <v-toolbar dense :color="titleColor" elevation="0">
                    <v-toolbar-title class="white--text">{{ title }}</v-toolbar-title>
                    <v-spacer></v-spacer>
                    <v-btn icon dark @click="$emit('close')">
                        <v-icon>close</v-icon>
                    </v-btn>
                </v-toolbar>
            </v-card-title>

            <v-card-title class="pa-0">
                <v-stepper non-linear flat v-if="pagination" style="width:100%; height:48px; background:var(--v-toolbar-base)" v-model="page">
                    <v-stepper-header>
                        <template v-for="page in pagination-1">
                            <v-stepper-step editable :step="page"></v-stepper-step>
                            <v-divider></v-divider>
                        </template>
                        <v-stepper-step editable :step="pagination"></v-stepper-step>
                    </v-stepper-header>
                </v-stepper>
            </v-card-title>

            <v-card-text class="pa-4" v-if="action=='delete'">
                Вы точно хотите удалить эту запись?
                <v-card-actions class="mt-4 pa-0">
                <v-btn :color="buttons.left.color" @click="$emit('close')">
                    <v-icon left>{{ buttons.left.icon }}</v-icon> {{ buttons.left.text }}
                </v-btn>
                <v-spacer></v-spacer>
                <v-btn :color="buttons.right.color" @click="$emit('save', {id:rowId})">
                    <v-icon left>{{ buttons.right.icon }}</v-icon> {{ buttons.right.text }}
                </v-btn>
                </v-card-actions>
            </v-card-text>

            <v-card-text class="pa-0" v-else>
                <mdp-edit-form
                    ref="mdpEditForm"
                    :action="action"
                    :tableName="tableName"
                    :tableInfo="tableInfo"
                    :rowId="rowId"
                    :buttons="buttons"
                    :page="page"
                    @close="$emit('close')"
                    @change="can_save = true"
                    @setpagination="pagination = $event; page = 1;"
                    :visible="visible"
                    @save="save"
                >
                </mdp-edit-form>
            </v-card-text>


            <v-divider></v-divider>
            <v-card-actions v-if="action!='delete'" style="background:var(--v-toolbar-base)">
                <v-btn v-if="pagination && page>1" class="mx-2" color="primary" outlined @click="page = page - 1">
                    <v-icon left>mdi-chevron-double-left</v-icon> Назад
                </v-btn>
                <v-btn v-else class="mx-2" :color="buttons.left.color"  @click="$emit('close')">
                    <v-icon left>{{ buttons.left.icon }}</v-icon> {{ buttons.left.text }}
                </v-btn>
                <v-spacer></v-spacer>
                <v-btn v-if="pagination && page<pagination" class="mx-2" color="primary" outlined @click="page = page + 1">
                    Далее <v-icon right>mdi-chevron-double-right</v-icon>
                </v-btn>
                <v-btn v-else class="mx-2" :color="buttons.right.color"  :disabled="!can_save" @click="$refs.mdpEditForm.save()">
                    <v-icon left>{{ buttons.right.icon }}</v-icon> {{ buttons.right.text }}
                </v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script>
import { mapActions } from "vuex";

export default {
    components: {
        "mdp-edit-form": () => import("./editForm.vue"),
    },

    props: {
        visible: { type: Boolean, default: false },
        titleColor: { type: String, default: "primary" },
        title: { type: String, default: "Редактирование записи" },
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

    data() {
        return {
            pagination: false,
            page: 1,
            can_save: false,
        };
    },

    watch: {
        visible() {},
    },

    mounted() {
    },
    beforeDestroy() {
    },

    methods: {
        save(row) {
            this.$emit("save", row);
        },
    },
};
</script>

<style>
.v-stepper__header {
    height: 48px !important;
}
.v-stepper__step {
   padding: 2px 24px !important;
}
</style>