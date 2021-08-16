<template>
    <div id="field-date" class="mb-1">
        <label class="v-label">{{ label }}</label>
        <div class="d-flex">
            <v-text-field
                :value="formatDate(date)"
                prepend-inner-icon="mdi-calendar"
                readonly
                @mouseup="open()"
                dense
                :hide-details="hideDetails"
                :outlined="outlined"
                :rules="rules"
                :style="[type == 'date' ? '' : 'max-width:140px']"
                :clearable="clearable"
                :disabled="disabled"
                @click:clear.prevent="
                    date = '';
                    changeDate();
                "
            ></v-text-field>

            <v-text-field
                v-model="time"
                v-if="type == 'dateTime' || type == 'timestamp'"
                prepend-inner-icon="mdi-clock-time-seven-outline"
                type="time"
                @input="changeDate()"
                dense
                :hide-details="hideDetails"
                :outlined="outlined"
                :rules="rules"
                class="ml-4"
                style="max-width:150px"
                step="2"
            ></v-text-field>
        </div>

        <v-dialog ref="dialog" v-model="date_dialog" :return-value.sync="date" width="300px">
            <v-date-picker v-model="date" scrollable :first-day-of-week="1" :min="min" :max="max" locale="ru-RU">
                <v-btn color="secondary" @click="date_dialog = false">
                    Отмена
                </v-btn>
                <v-spacer></v-spacer>
                <v-btn color="primary" @click="changeDate(date)">
                    Выбрать
                </v-btn>
            </v-date-picker>
        </v-dialog>
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
        hideDetails: { type: Boolean, default: false },
        disabled: { type: Boolean, default: false },
        rules: { type: Array, default: () => [] },
    },
    data() {
        return {
            date_dialog: false,
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
                this.date = this.$moment().format("YYYY-MM-DD");
                return;
            }
            this.date = this.value.substr(0, 10);
            this.time = this.value.substr(11);
            //console.log("date", this.date);
        },
        formatDate(date) {
            if (date && date.indexOf("-") == -1) return date;

            let newdate = "";
            try {
                newdate = date.substr(8, 2) + "." + date.substr(5, 2) + "." + date.substr(0, 4);
                //if (date.length > 12) newdate += " " + date.substr(11);
            } catch (e) {}
            return newdate;
        },

        changeDate(date) {
            if (date) {
                this.$refs.dialog.save(date);
                this.date = date;
            }
            let datetime = this.date;
            if (this.type == "dateTime" || this.type == "timestamp") datetime += " " + this.time;
            this.$emit("input", datetime);
        },
    },
};
</script>

<style></style>
