<template>
    <div>
        <v-dialog v-model="date_dialog" width="300px">
            <v-date-picker
                v-model="date_selected"
                locale="ru-RU"
                :first-day-of-week="1"
                :weekday-format="weekdayFormat"
                @click:date="changeDate"
                @click:month="changeDate"
                :range="isRange || isWeek"
                :type="isMonth ? 'month' : 'date'"
                scrollable
            ></v-date-picker>
        </v-dialog>
        <v-row justify="center">
            <div class="title text-center">
                <div class="d-flex">
                    <v-btn class="mt-1" icon @click="prev" v-if="showButton"><v-icon>mdi-arrow-left-thick</v-icon></v-btn>
                    <v-card outlined class="mx-1 px-2 py-1 elevation-1 d-flex" @click="date_dialog = true">
                        <v-icon v-if="!showHome" color="indigo" style="margin:-2px 5px 0 0;">mdi-calendar-clock</v-icon>
                        {{ date_text }}
                        <div v-if="showHome">
                            <v-btn icon small style="margin:-4px 0 0 2px;" @click.stop="setToday"><v-icon color="indigo">mdi-calendar-cursor</v-icon></v-btn>
                        </div>
                    </v-card>
                    <v-btn class="mt-1" icon @click="next" v-if="showButton"><v-icon>mdi-arrow-right-thick</v-icon></v-btn>
                </div>
            </div>
        </v-row>
    </div>
</template>

<script>
import moment from "moment";

export default {
    components: {},
    props: {
        value: { type: String, default: "" },
        isWeek: { type: Boolean, default: false }, //Выбрать неделю
        isMonth: { type: Boolean, default: false }, //Выбрать месяц
        isRange: { type: Boolean, default: false }, //Выбрать диапазон
        showButton: { type: Boolean, default: true }, //Показывать кнопки лево/право
        showHome: { type: Boolean, default: true }, //Показывать кнопки Сегодня
    },
    data() {
        return {
            date_dialog: false,
            date_selected: ["2020-05-04", "2020-05-10"],
            weekdays: ["Вс", "Пн", "Вт", "Ср", "Чт", "Пт", "Сб"],
        };
    },
    computed: {
        date_text() {
            if (this.isRange || this.isWeek) {
                return moment(this.date_selected[0]).format("DD.MM.YYYY") + " - " + moment(this.date_selected[1]).format("DD.MM.YYYY");
            }
            if (this.isMonth) {
                return moment(this.date_selected).format("MMMM YYYY");
            }
            return moment(this.date_selected).format("DD MMMM YYYY");
        },
    },
    watch: {
        value() {
            this.init();
        },
    },
    created() {
        moment.locale("ru");
    },
    mounted() {
        this.init();
    },

    methods: {
        weekdayFormat(val) {
            return this.weekdays[moment(val).format("d")];
        },

        init() {
            let dt = this.value;
            if (!dt || dt.length < 10) dt = moment().format("YYYY-MM-DD");
            if (this.isRange) this.date_selected = [dt, dt];
            if (!this.isRange && !this.isWeek) this.date_selected = dt;
            this.changeDate(dt);
        },

        prev() {
            if (this.isRange || this.isWeek) {
                let dt = this.date_selected[0];
                this.changeDate(
                    moment(dt)
                        .add(-7, "days")
                        .format("YYYY-MM-DD")
                );
                return;
            }
            let dt = this.date_selected;
            if (this.isMonth) {
                this.changeDate(
                    moment(dt)
                        .add(-1, "month")
                        .format("YYYY-MM-DD")
                );
                return;
            }
            this.changeDate(
                moment(dt)
                    .add(-1, "days")
                    .format("YYYY-MM-DD")
            );
        },
        next() {
            if (this.isRange || this.isWeek) {
                let dt = this.date_selected[0];
                this.changeDate(
                    moment(dt)
                        .add(+7, "days")
                        .format("YYYY-MM-DD")
                );
                return;
            }
            let dt = this.date_selected;
            if (this.isMonth) {
                this.changeDate(
                    moment(dt)
                        .add(+1, "month")
                        .format("YYYY-MM-DD")
                );
                return;
            }
            this.changeDate(
                moment(dt)
                    .add(+1, "days")
                    .format("YYYY-MM-DD")
            );
        },

        setToday() {
            let dt = moment().format("YYYY-MM-DD");
            if (this.isRange) this.date_selected = [dt, dt];
            if (!this.isRange && !this.isWeek) this.date_selected = dt;
            this.changeDate(dt);
        },

        changeDate(val) {
            if (!this.isRange && !this.isWeek) this.date_selected = val;

            if (this.isWeek) {
                if (moment(val).format("d") != 1) {
                    for (let i = 0; i < 7; i++) {
                        val = moment(val)
                            .add(-1, "days")
                            .format("YYYY-MM-DD");
                        if (moment(val).format("d") == 1) break;
                    }
                }
                this.date_selected = [
                    val,
                    moment(val)
                        .add(6, "days")
                        .format("YYYY-MM-DD"),
                ];
            }
            if (this.isRange && this.date_selected.length < 2) {
                return;
            }

            this.date_dialog = false;

            if (this.isRange || this.isWeek) {
                this.$emit("input", this.date_selected[0]);
            } else {
                this.$emit("input", this.date_selected);
            }
        },
    }, //methods
};
</script>

<style></style>
