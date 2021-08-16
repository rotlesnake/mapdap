<template>
    <div id="field-date" class="mb-1">
        <label class="v-label">{{ label }}</label>
        <div class="d-flex">
            <v-text-field
                :value="formatDate(date)"
                prepend-inner-icon="mdi-calendar"
                readonly
                @mouseup="date_dialog = true"
                dense
                :hide-details="hideDetails"
                :outlined="outlined"
                :rules="rules"
                style="max-width:140px"
            ></v-text-field>
            <v-text-field
                :value="formatDate(date)"
                class="ml-4"
                prepend-inner-icon="mdi-calendar"
                readonly
                @mouseup="date_dialog = true"
                dense
                :hide-details="hideDetails"
                :outlined="outlined"
                :rules="rules"
                style="max-width:140px"
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
        label: { type: String, default: null },
        value: { required: true },
        min: { type: String, default: null },
        max: { type: String, default: null },
        outlined: { type: Boolean, default: false },
        hideDetails: { type: Boolean, default: false },
        rules: { type: Array, default: () => [] },
    },
    data() {
        return {
            date_dialog: false,
            date: "",
        };
    },
    watch: {
        value() {
            this.refresh();
        },
    },
    mounted() {
        this.date = this.value;
        this.refresh();
    },

    methods: {
        refresh() {},
        formatDate(date) {
            if (date && date.indexOf("-") == -1) return date;

            let newdate = "";
            try {
                newdate = date.substr(8, 2) + "." + date.substr(5, 2) + "." + date.substr(0, 4);
                if (date.length > 12) newdate += " ".date.substr(12);
            } catch (e) {}
            return newdate;
        },
        changeDate(date) {
            this.$refs.dialog.save(date);
            this.date = date;
            this.$emit("input", this.date);
            this.refresh();
        },
    },
};
</script>

<style></style>
