<template>
    <div class="mt-2">
        <v-menu offset-y :value="kladr_items.length > 0" max-height="250px">
            <template #activator>
                <v-text-field
                    v-model="text"
                    placeholder="Город Улица Дом"
                    @input="onTextInput"
                    :loading="isLoading"
                    :loader-height="5"
                    :hide-details="hideDetails"
                    :label="label"
                    :outlined="outlined"
                    :dense="dense"
                    :clearable="clearable"
                    :disabled="disabled"
                />
            </template>
            <v-list>
                <v-list-item v-for="(item, index) in kladr_items" :key="index" @click="selectAddress(item)">
                    <v-list-item-title>{{ item.text }}</v-list-item-title>
                </v-list-item>
            </v-list>
        </v-menu>
    </div>
</template>

<script>
import common from "@/components/common.js";
export default {
    layout: "empty",
    computed: {},
    props: {
        value: { type: String, default: "" },
        label: { type: String, default: null },
        clearable: { type: Boolean, default: true },
        outlined: { type: Boolean, default: false },
        dense: { type: Boolean, default: false },
        hideDetails: { type: Boolean, default: false },
        disabled: { type: Boolean, default: false },
    },

    data() {
        return {
            text: this.value,
            kladr_items: [],
            isLoading: false,
            selected_item: null,
        };
    },

    watch:{
        value(){
            this.text = this.value;
        },
    },

    created() {
        this.onTextInput = common.debounce(this.onInput, 500);
    },

    methods: {
        onInput() {
            if (this.text < 4) {
                this.selected_item = null;
                this.$emit("input", "");
                return;
            }
            if (this.selected_item) {
                this.$emit("input", this.text);
                return;
            }

            this.isLoading = true;
            this.kladr_items = [];
            this.$axios.get("https://yy7.ru/kladr/?search=" + this.text).then((response) => {
                this.isLoading = false;
                this.kladr_items = response.data;
            });
        },

        selectAddress(item) {
            this.selected_item = item;
            this.kladr_items = [];
            this.text = item.text;
            this.$emit("input", item.text);
        },
    },
};
</script>
