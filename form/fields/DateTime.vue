<template>
    <div id="field-date" class="mb-1">
        <div class="d-flex">
            <v-menu ref="date_dialog" v-model="date_dialog" :close-on-content-click="false" transition="scale-transition" offset-y :nudge-right="-150" max-width="290px" min-width="auto">
                <template v-slot:activator="{ on, attrs }">
                    <v-text-field
                        v-model="dateEdt"
                        :hide-details="hideDetails"
                        :label="label"
                        :name="name"
                        :outlined="outlined"
                        :dense="dense"
                        :rules="rules"
                        :style="[type == 'date' ? '' : 'max-width:140px']"
                        :clearable="clearable"
                        :disabled="disabled"
                        placeholder="дд.мм.гггг"
                        persistent-placeholder
                        append-icon="mdi-calendar"
                        @click:append="open"
                        @click:clear="clearDate"
                        @blur="dateEditFinish"
                        @keyup.enter="dateEditFinish"
                        ref="masked_datetime"
                    ></v-text-field>
                </template>

                <v-date-picker
                    v-model="date"
                    no-title
                    :min="min"
                    :max="max"
                    :events="events"
                    :event-color="eventColor"
                    :first-day-of-week="1"
                    :weekday-format="weekdayFormat"
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
        name: { type: String, default: null },
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
        customStyle: { type: Object, default:()=>({})  },
        events: { type: Array, default: () => [] },
        eventColor: { type: String, default: "" },
    },
    data() {
        return {
            weekdays: ["Вс", "Пн", "Вт", "Ср", "Чт", "Пт", "Сб"],
            date_dialog: false,
            dateEdt: "",
            localValue: "",
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
        Inputmask({ mask: "99.99.9999", autoUnmask: false, placeholder:"дд.мм.гггг" }).mask(el);
    },

    methods: {
        open(){
            if (this.disabled) return;
            if (!this.date || this.date=="0000-00-00") {
                this.date = moment().format("YYYY-MM-DD");
                this.time = "00:00:00";
                this.dateEditFinish();
            }
            this.date_dialog = true;
            setTimeout(()=>{
               Object.keys(this.customStyle).forEach(key=>{
                   this.$refs.date_dialog.$refs.content.style[key] = this.customStyle[key];
               });
            },100);
        },
        refresh() {
            if (!this.value || this.value.length < 10) {
                this.date = "";
                this.time = "00:00:00";
                this.dateEditFinish();
                return;
            }
            if (this.localValue == this.value) return;

            this.localValue = this.value;
            this.date = this.localValue.substr(0, 10);
            this.time = this.localValue.substr(11);
            this.dateEdt = "";
            this.dateEditFinish();
        },

        weekdayFormat(val) {
            return this.weekdays[moment(val).format("d")];
        },

        dateToSql(dt) {
            dt = dt.replace(/_/g,"");
            dt = dt.replace(/г/g,"");
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
            if (this.dateEdt == this.dateToDate(this.date)) return;
            if (this.dateEdt) this.date = this.dateToSql(this.dateEdt);
            if (this.min && this.date < this.min) this.date = "";
            if (this.max && this.date > this.max) this.date = "";
            this.dateEdt = this.dateToDate(this.date);
            this.changeDate();
        },
        clearDate(){
            this.date = "";
            this.time = "";
            this.dateEdt = "";
            this.changeDate();
        },


        changeDate() {
            let datetime = this.date;
            if (datetime && (this.type == "dateTime" || this.type == "timestamp")) datetime += " " + this.time;
            this.$emit("input", datetime);
        },
    },
};
</script>

<style></style>
