<template>
    <v-dialog v-model="visible" scrollable persistent max-width="990px">
        <v-card>
            <v-toolbar dense dark color="primary" elevation="0">
                <v-toolbar-title>Видимость колонок</v-toolbar-title>
                <v-spacer></v-spacer>
                <v-btn icon dark @click="$emit('close')">
                    <v-icon>close</v-icon>
                </v-btn>
            </v-toolbar>

            <v-card-text style="padding:5px 20px 10px 20px;">
                <v-form>
                    <template v-for="head_item in columns">
                        <v-checkbox
                            :key="head_item.id"
                            v-model="head_item.hidden"
                            off-icon="check_circle_outline"
                            on-icon="highlight_off"
                            color="#757575"
                            :class="{ 'as-disabled': head_item.hidden, 'as-enabled': !head_item.hidden }"
                            :label="head_item.text"
                            hide-details
                        ></v-checkbox>
                        <v-divider :key="head_item.id"></v-divider>
                    </template>
                </v-form>
            </v-card-text>

            <v-divider></v-divider>
            <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn class="mx-2" color="primary" @click="$emit('close')">
                    <v-icon left>close</v-icon> Закрыть
                </v-btn>
                <v-spacer></v-spacer>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script>
import { mapActions } from "vuex";

export default {
    components: {},

    props: {
        visible: { type: Boolean, default: false },
        value: { type: Array, default: null },
    },

    data() {
        return {
            columns: this.value,
        };
    },

    watch: {
        visible() {},
        value() {
            this.columns = this.value;
        },
    },

    methods: {
        save() {
            this.$emit("save", this.columns);
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
