<template>
    <v-dialog v-model="visible" scrollable persistent max-width="990px">
        <v-card>
            <v-card-title class="pa-0">
                <v-toolbar dense :color="titleColor" elevation="0">
                    <v-toolbar-title class="white--text">{{ title }}</v-toolbar-title>
                    <v-spacer></v-spacer>
                    <v-btn icon dark @click="$emit('close')">
                        <v-icon>close</v-icon>
                    </v-btn>
                </v-toolbar>
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
                <mdp-edit-form :action="action" :tableName="tableName" :tableInfo="tableInfo" :rowId="rowId" :buttons="buttons" @close="$emit('close')" :visible="visible" @save="save">
                </mdp-edit-form>
            </v-card-text>
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
        return {};
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
