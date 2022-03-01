<template>
    <div id="field-date" class="mb-1">
        <div class="d-flex">
            <v-menu ref="date_dialog" v-model="date_dialog" :close-on-content-click="false" transition="scale-transition" offset-y max-width="290px" min-width="auto">
                <template v-slot:activator="{ on, attrs }">
                    <v-text-field
                        v-model="dateEdt"
                        :hide-details="hideDetails"
                        :label="label"
                        :outlined="outlined"
                        :dense="dense"
                        :rules="rules"
                        :style="[type == 'date' ? '' : 'max-width:140px']"
                        :clearable="clearable"
                        :disabled="disabled"
                        placeholder="ДД.ММ.ГГГГ"
                        pre-pend-inner-icon="mdi-calendar"
                        append-icon="mdi-calendar"
                        @click:append="date_dialog = true"
                        @blur="dateEditFinish"
                        ref="masked_datetime"
                    ></v-text-field>
                </template>

                <v-date-picker
                    v-model="date"
                    no-title
                    :min="min"
                    :max="max"
                    @input="
                        dateEdt = dateToDate($event);
                        changeDate();
                        date_dialog = false;
                    "
                ></v-date-picker>
            </v-menu>

            <v-text-field
                v-model="time"
                v-if="type == 'dateTime' || type == 'timestamp'"
                prepend-inner-icon="mdi-clock-time-seven-outline"
                type="time"
                label="Время"
                @input="changeDate()"
                :hide-details="hideDetails"
                :outlined="outlined"
                :dense="dense"
                :rules="rules"
                class="ml-3"
                style="max-width:150px"
                :step="step"
            ></v-text-field>
        </div>

    </div>
</template>

<script>
import Inputmask from "inputmask";
import moment from "moment";
export default {
    props: {
        type: { type: String, default: "date" },
        label: { type: String, default: null },
        value: { required: true },
        min: { type: String, default: null },
        max: { type: String, default: null },
        clearable: { type: Boolean, default: true },
        outlined: { type: Boolean, default: false },
        dense: { type: Boolean, default: false },
        hideDetails: { type: Boolean, default: false },
        disabled: { type: Boolean, default: false },
        rules: { type: Array, default: () => [] },
        step: { type: String, default: "0" },
    },
    data() {
        return {
            date_dialog: false,
            dateEdt: "",
            date: "",
            time: "",
        };
    },
    watch: {
        value() {
            this.refresh();
        },
    },
    mounted() {
        this.refresh();

        let el = this.$refs.masked_datetime.$refs.input;
        Inputmask({ mask: "99.99.9999", autoUnmask: false }).mask(el);
    },

    methods: {
        open(){
            if (this.disabled) return;
            this.date_dialog = true;
        },
        refresh() {
            if (!this.value || this.value.length < 10) {
                this.date = "";
                this.time = "00:00:00";
                this.dateEditFinish();
                return;
            }
            this.date = this.value.substr(0, 10);
            this.time = this.value.substr(11);
            this.dateEditFinish();
        },

        dateToSql(dt) {
            dt = dt.replace(/_/g,"");
            let dta = dt.split(".");
            if (!dt) return "";
            if (dt.length < 10 && dta[2]) dt = dta[0]+"."+String(dta[1])+"."+(parseInt(2000)+parseInt(dta[2]));
            if (dt.substr(0, 5).indexOf("-") > -1) return dt;
            let val = moment(dt.substr(6, 4) + "-" + dt.substr(3, 2) + "-" + dt.substr(0, 2)).format("YYYY-MM-DD");
            if (val=="Invalid date") return "";
            return val;
        },
        dateToDate(dt) {
            if (!dt || dt.length < 10) return "";
            if (dt.substr(0, 3).indexOf(".") > -1) return dt;
            let val = moment(dt).format("DD.MM.YYYY");
            if (val=="Invalid date") return "";
            return val;
        },
        dateEditFinish() {
            if (this.dateEdt) this.date = this.dateToSql(this.dateEdt);
            if (this.min && this.date < this.min) this.date = "";
            if (this.max && this.date > this.max) this.date = "";
            this.dateEdt = this.dateToDate(this.date);
            this.changeDate();
        },


        changeDate() {
            let datetime = this.date;
            if (this.type == "dateTime" || this.type == "timestamp") datetime += " " + this.time;
            this.$emit("input", datetime);
        },
    },
};
</script>

<style></style>
