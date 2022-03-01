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
                        prepend-inner-icon="mdi-calendar"
                        append-icon="mdi-calendar"
                        @click:append="date_dialog = true"
                        @blur="dateEditFinish"
                        @keyup="dateInputMask_up"
                        @keydown="dateInputMask_down"

                    ></v-text-field>
                </template>

                <v-date-picker
                    v-model="date"
                    no-title
                    :min="min"
                    :max="max"
                    @input="
                        dateEdt = dateToDate(date);
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
    },

    methods: {
        open(){
            if (this.disabled) return;
            this.date_dialog = true;
        },
        refresh() {
            if (!this.value) {
                this.date = "";
                this.time = "00:00:00";
                return;
            }
            this.date = this.value.substr(0, 10);
            this.time = this.value.substr(11);
        },

        dateInputMask_up(e) {
            let v = e.target.value;
            if (e.keyCode != 8) {
                if (v.match(/^\d{2}$/) !== null) {
                    e.target.value = v + ".";
                } else if (v.match(/^\d{2}.\d{2}$/) !== null) {
                    e.target.value = v + ".";
                }
            }
            if (e.keyCode == 13) {
                this.dateEditFinish();
            }
            e.target.value = e.target.value.replace(/\s/g, "");
        },
        dateInputMask_down(e) {
            let len = e.target.value.length;
            if (
                e.keyCode == 46 ||
                e.keyCode == 8 ||
                e.keyCode == 9 ||
                e.keyCode == 27 ||
                // Разрешаем: Ctrl+A
                (e.keyCode == 65 && e.ctrlKey === true) ||
                // Разрешаем: home, end, влево, вправо
                (e.keyCode >= 35 && e.keyCode <= 39)
            ) {
                // Ничего не делаем
                return;
            } else {
                // Запрещаем все, кроме цифр на основной клавиатуре, а так же Num-клавиатуре
                if ((e.keyCode < 48 || e.keyCode > 57) && (e.keyCode < 96 || e.keyCode > 105)) {
                    e.preventDefault();
                }
            }
            if (len >= 10) {
                e.preventDefault();
            }
        },

        dateToSql(dt) {
            if (!dt || dt.length < 10) {
                let date = new Date();
                date = date.getFullYear() + "-" + ("00" + (date.getMonth() + 1)).slice(-2) + "-" + ("00" + date.getDate()).slice(-2);
                return date;
            }
            if (dt.substr(0, 5).indexOf("-") > -1) return dt;
            return dt.substr(6, 4) + "-" + dt.substr(3, 2) + "-" + dt.substr(0, 2);
        },
        dateToDate(dt) {
            if (!dt || dt.length < 10) {
                let date = new Date();
                date = ("00" + date.getDate()).slice(-2) + "." + ("00" + (date.getMonth() + 1)).slice(-2) + "." + date.getFullYear();
                return date;
            }
            if (dt.substr(0, 3).indexOf(".") > -1) return dt;
            return dt.substr(8, 2) + "." + dt.substr(5, 2) + "." + dt.substr(0, 4);
        },
        dateEditFinish() {
            this.date = this.dateToSql(this.dateEdt);
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
