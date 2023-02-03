<template>
    <div class="mt-2" v-if="options">
        <v-autocomplete
            ref="combobox"
            :menu-props="{ bottom: true, offsetY: true }"
            :items="rest_items"
            v-model="localValue"
            :search-input.sync="autocompleteSearch"
            :multiple="options.multiple"
            chips
            @change="selectItem"
            v-bind="options"
            :rules="rules"
            :disabled="disabled"
        >
            <template v-slot:item="{ item }">
                <div class="px-2">
                    <div class="title">{{ item.text }}</div>
                    <div class="caption">{{ item.desc }}</div>
                </div>
            </template>
            <template v-slot:selection="{ item, index }">
                <v-chip :small="options.dense" close @click:close="removeItem($event,item,index)">
                    <span>{{ item.text }}</span>
                </v-chip>
            </template>
        </v-autocomplete>
    </div>
</template>

<script>
import common from "@/components/common.js";
export default {
    props: {
        value: { type: String, default: "" },
        options: { type: Object, default: null },
        rules: { type: Array, default: () => [] },
        disabled: { default: false },
    },

    data() {
        return {
            autocompleteSearch: null,
            localValue: [],
            rest_items: [],
            isLoading: false,
            selected_item: null,
        };
    },
    watch: {
        value() {
            this.changeValue();
        },
        autocompleteSearch(val) {
            this.onTextInput(val);
        },
    },

    created() {
        this.onTextInput = common.debounce(this.onInput, 500);
    },
    mounted() {
        this.changeValue();
    },

    methods: {
        changeValue() {
            if (this.options && this.value && this.localValue != this.value) {
                if (this.options.multiple) {
                    if (typeof this.value != "object") { this.localValue = [this.value]; } else { this.localValue = this.value; }
                } else {
                    if (typeof this.value == "object") { this.localValue = this.value[0]; } else { this.localValue = this.value; }
                }

                this.$axios.post(this.options.rest.url, {value:this.localValue}).then((response) => {
                    this.isLoading = false;
                    this.rest_items = response.data;
                });
            }
        },

        onInput(val) {
            this.isLoading = true;
            this.$axios.post(this.options.rest.url, {text:val, value:this.localValue}).then((response) => {
                this.isLoading = false;
                this.rest_items = response.data;
            });
        },

        removeItem(evt, item, index) {
            if (isNaN(parseInt(index))) return;
            if (!this.options.multiple) { this.localValue = null; return this.$emit("input", this.localValue); }
            this.localValue.splice(index, 1);
            this.$emit("input", this.localValue);
        },

        selectItem(item) {
            this.localValue = item;
            this.$emit("input", this.localValue);
            this.$nextTick(()=>{ this.$refs.combobox.isMenuActive = false; });
        },
    },
};
</script>
