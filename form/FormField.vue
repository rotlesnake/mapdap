<template>
    <div v-if="options && options.name" :style="(options.formWidth ? 'width:calc('+options.formWidth+' - 12px); margin:0 6px;' : 'width:100%;')">
        <template v-if="options.caption && vifCalc()">
            <div class="title mb-1">{{ options.caption }}</div>
        </template>

        <!-- SIMPLE !-->
        <template v-if="options.type == 'string' && vifCalc()">
            <v-text-field
                v-model="valueLocal"
                v-bind="options"
                @input="onInput"
                :rules="fieldRules"
                :disabled="disabled"
                :hidden="false"
                :name="name"
                @click:append="showTooltip($event, options)"
            ></v-text-field>
        </template>

        <template v-if="options.type == 'text' && vifCalc()">
            <v-textarea
                v-model="valueLocal"
                v-bind="options"
                @input="onInput"
                :rules="fieldRules"
                :disabled="disabled"
                :hidden="false"
                :name="name"
                @click:append="showTooltip($event, options)"
            ></v-textarea>
        </template>

        <template v-if="(options.type == 'integer' || options.type == 'bigInteger') && vifCalc()">
            <v-text-field
                v-model="valueLocal"
                type="number"
                v-bind="options"
                @input="onInput"
                :rules="fieldRules"
                :disabled="disabled"
                :hidden="false"
                :name="name"
                @click:append="showTooltip($event, options)"
            ></v-text-field>
        </template>
        <template v-if="options.type == 'float' || options.type == 'double' && vifCalc()">
            <v-text-field
                v-model="valueLocal"
                type="number"
                v-bind="options"
                @input="onInput"
                :rules="fieldRules"
                :disabled="disabled"
                :hidden="false"
                :name="name"
                @click:append="showTooltip($event, options)"
            ></v-text-field>
        </template>

        <template v-if="options.type == 'password' && vifCalc()">
            <v-text-field
                v-model="valueLocal"
                v-bind="options"
                @input="onInput"
                :rules="passwordRules"
                counter
                :append-icon="show_password ? 'visibility' : 'visibility_off'"
                @click:append="show_password = !show_password"
                :type="show_password ? 'text' : 'password'"
                :disabled="disabled"
                :hidden="false"
                :name="name"
            ></v-text-field>
        </template>

        <template v-if="options.type=='checkBox' && vifCalc()">
            <v-switch  v-model="valueLocal" 
                    v-bind="options"
                    :label="`${options.label} (${valueLocal==1?'Да':'Нет'})`" :true-value="1" :false-value="0"
                    @change="onInput"
                    :rules="fieldRules"
                    :disabled="disabled"
                    :hidden="false"
                    :name="name"
            ></v-switch>
        </template>

        <!-- IMAGES FILES !-->
        <template v-if="options.type == 'images' && vifCalc()">
            <field-images
                v-model="valueLocal"
                v-bind="options"
                @input="onInput"
                :disabled="disabled"
                :hidden="false"
                :name="name"
            ></field-images>
        </template>
        <template v-if="options.type == 'files' && vifCalc()">
            <field-files
                v-model="valueLocal"
                v-bind="options"
                @input="onInput"
                :disabled="disabled"
                :hidden="false"
                :name="name"
            ></field-files>
        </template>

        <!-- SELECTABLE !-->
        <template v-if="options.type == 'select' && vifCalc()">
            <v-select
                :items="items"
                v-model="valueLocal"
                chips
                v-bind="options"
                :rules="fieldRules"
                item-value="value"
                item-text="text"
                @input="onInput"
                :disabled="disabled"
                :hidden="false"
                :name="name"
            >
            <template v-slot:selection="{ item, index }">
                <v-chip :small="options.dense">
                    <span>{{ item.text }}</span>
                </v-chip>
            </template>
            <template v-slot:append v-if="options['append-icon']">
                <v-icon @click.prevent="showTooltip($event, options)">{{options['append-icon']}}</v-icon>
                <v-icon>mdi-menu-down</v-icon>
            </template>
            </v-select>
        </template>

        <template v-if="options.type == 'linkTable' && vifCalc()">
            <field-select-from-table
                :value="valueLocal"
                :items="items"
                :row="row"
                :options="options"
                :rules="fieldRules"
                @change="linkTableChange"
                :disabled="disabled"
                :hidden="false"
                :name="name"
                :clickAppend="showTooltip"
            ></field-select-from-table>
        </template>


        <template v-if="options.type == 'json' && vifCalc()">
            <json-table
                v-model="valueLocal"
                :row="row"
                :options="options"
                :rules="fieldRules"
                @input="onInput"
                :disabled="disabled"
                :hidden="false"
                :name="name"
            ></json-table>
        </template>


        <!-- PICKERS !-->
        <template v-if="(options.type == 'date' || options.type == 'dateTime' || options.type == 'timestamp') && vifCalc()">
            <field-date
                v-model="valueLocal"
                v-bind="options"
                @input="onInput"
                :disabled="disabled"
                :hidden="false"
                :name="name"
            ></field-date>
        </template>
        <template v-if="options.type == 'dateRange'">
            <field-date-range
                v-model="valueLocal"
                v-bind="options"
                @input="onInput"
                :disabled="disabled"
                :hidden="false"
                :name="name"
            ></field-date-range>
        </template>

        <template v-if="options.type == 'time' && vifCalc()">
            <v-text-field
                v-model="valueLocal"
                v-bind="options"
                prepend-inner-icon="mdi-clock-time-seven-outline"
                type="time"
                @input="onInput"
                :rules="fieldRules"
                :disabled="disabled"
                :hidden="false"
                :name="name"
                style="max-width:300px"
                @click:append="showTooltip($event, options)"
                clearable
                @click:clear.prevent="onClear"
            ></v-text-field>
        </template>
        
        <template v-if="options.type == 'color' && vifCalc()">
            <field-color
                v-model="valueLocal"
                v-bind="options"
                @input="onInput"
                :rules="fieldRules"
                :disabled="disabled"
                :hidden="false"
                :name="name"
            ></field-color>
        </template>


        <v-menu
            v-model="popupTooltip.visible"
            :position-x="popupTooltip.x"
            :position-y="popupTooltip.y"
            absolute>
            <v-card v-html="popupTooltip.html" class="pa-2 primary--text">
            </v-card>
        </v-menu>
    </div>
</template>

<script>
import Vue from "vue";

export default {
    components: {
        "field-color": () => import("../LV/LvColorPicker.vue"),
        "field-images": () => import("./fields/Images.vue"),
        "field-files": () => import("./fields/Files.vue"),
        "field-select-from-table": () => import("./fields/SelectFromTable.vue"),
        "field-date": () => import("./fields/Date.vue"),
        "field-date-range": () => import("./fields/DateRange.vue"),
        "json-table": () => import("./fields/JsonTable.vue"),
    },
    props: {
        row: { required: true },
        value: { required: true },
        options: { type: Object, default: ()=>{} },
        name: { type: String, default: null },
        disabled: { type: Boolean, default: false },
    },
    watch: {
        options() {
            this.refresh();
        },
        name() {
            this.refresh();
        },
        value() {
            this.refresh();
        },
        row() {
            this.refresh();
        },
    },
    data() {
        return {
            valueLocal: "",
            items: [],
            itemsAll: [],
            items_laded: true,
            show_password: false,
            popupTooltip:{
                visible: false,
                x: 0,
                y: 0,
                html: "",
            },
            fieldRules: [],
            fileRules: [
                (files) =>
                    files.length == 0 ||
                    !files[0].size ||
                    (files[0] && files[0].size < 5 * 1024 * 1024) ||
                    "Размер файла должен быть не более 5 MB",
            ],
            passwordRules: [
                (v) =>
                    !v || (v && v.length < 1) || (v && v.length >= 8) || "Длина пароля должна быть не менее 8 символов",
            ],
        }; //return
    },
    created() {
        this.refresh();
    },

    methods: {
        showTooltip(evt, options) {
            evt.preventDefault();
            this.popupTooltip.visible = false;
            this.popupTooltip.x = evt.clientX;
            this.popupTooltip.y = evt.clientY;
            this.popupTooltip.html = options["append-icon-text"];
            this.$nextTick(() => {
                this.popupTooltip.visible = true;
            });
        },
        onClear() {
            this.$nextTick().then(() => {
                this.valueLocal=''; this.onInput('');
            });
        },
        onInput(value) {
            this.$emit("input", this.valueLocal);
            this.$emit("change", this.name, this.valueLocal, this.valueLocal);
            if (this.options.afterChange && this.options.afterChange.length>1) this.afterChangeField(this.name);
        },
        linkTableChange(values, items, rows) {
            this.valueLocal = values;
            this.items = items;
            this.row[this.options.name + "_values"] = items;
            this.row[this.options.name + "_text"] = items[0] ? items[0].text : "";
            this.$emit("input", values);
            this.$emit("change", this.name, values, items, rows);
            if (this.options.afterChange && this.options.afterChange.length>1) this.afterChangeField(this.name);
        },

        refresh() {
            this.valueLocal = this.value;
            this.fieldRules = this.stringToRules(this.options.rules);

            if (
                this.options.default &&
                this.options.default.length > 0 &&
                (!this.options.placeholder || this.options.placeholder.length == 0)
            ) {
                this.options.placeholder = this.options.default;
            }

            if (this.options.type == "select") {
                this.items = [];
                for (let item in this.options.items) {
                    this.items.push({ value: parseInt(item), text: this.options.items[item] });
                }
                this.valueLocal = parseInt(this.value) || 0;
            }
            if (this.options.type == "linkTable") {
                this.items = [];
                if (this.options.multiple) {
                    this.items = this.row[this.options.name + "_values"];
                } else {
                    this.items = [{ value: parseInt(this.value), text: this.row[this.options.name + "_text"] }];
                }
            }

            if (this.options.defaultFront && this.options.defaultFront.length>0 && !this.valueLocal) {
                this.valueLocal = this.options.defaultFront;
                if (this.options.type == "select") this.valueLocal = parseInt(this.options.defaultFront);
                if (this.options.type == "linkTable") { const val = this.options.defaultFront.split("."); this.items = [{ value: parseInt(val[0]), text: val[1] }]; this.valueLocal = [parseInt(val[0])]; }
                if (this.options.type == "date" && this.options.defaultFront == "now") this.valueLocal = this.$moment().format("YYYY-MM-DD");
                if (this.options.type == "dateTime" && this.options.defaultFront == "now") this.valueLocal = this.$moment().format("YYYY-MM-DD")+" 00:00:00";
            }

            this.$forceUpdate();
        },

        afterChangeField(fldName) {
            if (this.options.afterChange && this.options.afterChange.length>1) {
                let rules = this.options.afterChange;
                rules = rules.replace(/\[(.*?)\]/gi, (match, name) => {
                    if (!this.row[name]) return 0;
                    if (typeof this.row[name] == "object" && this.row[name].length==0) return 0;
                    if (typeof this.row[name] == "object" && rules.indexOf('IN(')>-1 ) return "["+this.row[name]+"]";
                    return this.row[name];
                });

                var thisContext = this;
                rules = rules.split(" ;; ");
                rules.forEach(e=>{
                    e = "let result = function(){ if ("+e.replace(' ? ', ') return ')+"; return null; }();";
                    let calc = "function IN(arr,val){ if (typeof arr != 'object') return arr==val;  return arr.indexOf(val)>-1; }; ";
                       calc += "function SET(fld,val,txt){ thisContext.$emit('changeRowField', fld, val, txt); }; "+e;
                    eval(calc);
                });

            }
        },

        vifCalc() {
            if (!this.options.vif || this.options.vif.length < 2) return true;
            let rules = this.options.vif;
            rules = rules.replace(/\[(.*?)\]/gi, (match, name) => {
                if (!this.row[name]) return 0;
                if (typeof this.row[name] == "object" && this.row[name].length==0) return 0;
                if (typeof this.row[name] == "object" && rules.indexOf('IN(')>-1 ) return "["+this.row[name]+"]";
                return this.row[name];
            });
            rules = "function IN(arr,val){ if (typeof arr != 'object') return arr==val;  return arr.indexOf(val)>-1; }; "+rules;

            return eval(rules);
        },
        stringToRules(rulestr) {
            if (rulestr instanceof Array) return rulestr;
            if (!rulestr) return [];
            if (rulestr.length < 9) return [];
            return eval(rulestr);
        },
    }, //methods:
};
</script>

<style></style>
