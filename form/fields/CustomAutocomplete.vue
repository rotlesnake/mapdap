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
                <v-chip :small="options.dense" :close="!disabled" @click:close="removeItem($event,item,index)">
                    <span>{{ item.text }}</span>
                </v-chip>
            </template>
        </v-autocomplete>
    </div>
</template>

<script>
import common from "../../common/common.js";
export default {
    props: {
        value: { required: false },
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
        if (this.value) {
           this.changeValue();
        } else {
           this.onInput("");
        }
    },

    methods: {
        changeValue() {
            if (this.options && this.value && this.localValue != this.value) {
                if (this.options.multiple) {
                    if (typeof this.value != "object") { this.localValue = [this.value]; } else { this.localValue = this.value; }
                } else {
                    if (typeof this.value == "object") { this.localValue = this.value[0]; } else { this.localValue = this.value; }
                }

                this.$axios.post(this.options.rest.url || this.options.rest, {value:this.localValue}).then((response) => {
                    this.isLoading = false;
                    this.rest_items = response.data;
                    if (this.rest_items) {
                        this.selectItem(this.localValue);
                    } else {
                        this.rest_items = [];
                    }
                });
            }
        },

        onInput(val) {
            this.isLoading = true;
            this.$axios.post(this.options.rest.url || this.options.rest, {text:val, value:this.localValue}).then((response) => {
                this.isLoading = false;
                this.rest_items = response.data;
                if (!this.rest_items) this.rest_items = [];
            });
        },

        removeItem(evt, item, index) {
            if (isNaN(parseInt(index))) return;
            if (!this.options.multiple) { this.localValue = null; return this.$emit("input", this.localValue); }
            this.localValue.splice(index, 1);
            this.$emit("input", this.localValue);
        },

        selectItem(selectedValue) {
            this.localValue = selectedValue;
            let items = this.rest_items.filter(e=> this.localValue == e.value);

            if (this.options.multiple) {
                items = this.rest_items.filter(e=> this.localValue.includes(e.value));
            }

            this.$emit("change", this.localValue, items, []);
            this.$nextTick(()=>{ this.$refs.combobox.isMenuActive = false; });
        },
    },
};
</script>
