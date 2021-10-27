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
                    <v-btn class="mt-1" :color="color" icon @click="prev" v-if="showButton"><v-icon>mdi-arrow-left-thick</v-icon></v-btn>
                    <v-card outlined :color="color" class="mx-1 px-2 py-1 elevation-1 d-flex justify-center" @click="date_dialog = true" :style="(width ? 'width:'+width+';' : '')">
                        <v-icon v-if="showIcon" color="indigo" style="margin:-2px 5px 0 0;">mdi-calendar-clock</v-icon>
                        {{ date_text }}
                        <div v-if="showHome">
                            <v-btn icon small style="margin:-4px 0 0 2px;" @click.stop="setToday"><v-icon color="indigo">mdi-calendar-cursor</v-icon></v-btn>
                        </div>
                    </v-card>
                    <v-btn class="mt-1" :color="color" icon @click="next" v-if="showButton"><v-icon>mdi-arrow-right-thick</v-icon></v-btn>
                </div>
            </div>
        </v-row>
    </div>
</template>

<script>
export default {
    components: {},
    props: {
        value: { type: String, default: "" },
        width: { type: String, default: "" },
        color: { type: String, default: "" },
        isWeek: { type: Boolean, default: false }, //Выбрать неделю
        isMonth: { type: Boolean, default: false }, //Выбрать месяц
        isRange: { type: Boolean, default: false }, //Выбрать диапазон
        showButton: { type: Boolean, default: true }, //Показывать кнопки лево/право
        showHome: { type: Boolean, default: true }, //Показывать кнопки Сегодня
        showIcon: { type: Boolean, default: true }, //Показывать кнопки Календаря
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
                return this.$moment(this.date_selected[0]).format("DD.MM.YYYY") + " - " + this.$moment(this.date_selected[1]).format("DD.MM.YYYY");
            }
            if (this.isMonth) {
                return this.$moment(this.date_selected).format("MMMM YYYY");
            }
            return this.$moment(this.date_selected).format("DD MMMM YYYY");
        },
    },
    watch: {
        value() {
            this.init();
        },
    },

    mounted() {
        this.init();
        this.changeDate();
    },

    methods: {
        weekdayFormat(val) {
            return this.weekdays[this.$moment(val).format("d")];
        },

        init() {
            let dt = this.value;
            if (!dt || dt.length < 10) dt = this.$moment().format("YYYY-MM-DD");
            if (this.isRange) this.date_selected = [dt, dt];
            if (!this.isRange && !this.isWeek) this.date_selected = dt;
            this.applyDate(dt);
        },

        prev() {
            if (this.isRange || this.isWeek) {
                let dt = this.date_selected[0];
                this.changeDate(
                    this.$moment(dt)
                        .add(-7, "days")
                        .format("YYYY-MM-DD")
                );
                return;
            }
            let dt = this.date_selected;
            if (this.isMonth) {
                this.changeDate(
                    this.$moment(dt)
                        .add(-1, "month")
                        .format("YYYY-MM-DD")
                );
                return;
            }
            this.changeDate(
                this.$moment(dt)
                    .add(-1, "days")
                    .format("YYYY-MM-DD")
            );
        },
        next() {
            if (this.isRange || this.isWeek) {
                let dt = this.date_selected[0];
                this.changeDate(
                    this.$moment(dt)
                        .add(+7, "days")
                        .format("YYYY-MM-DD")
                );
                return;
            }
            let dt = this.date_selected;
            if (this.isMonth) {
                this.changeDate(
                    this.$moment(dt)
                        .add(+1, "month")
                        .format("YYYY-MM-DD")
                );
                return;
            }
            this.changeDate(
                this.$moment(dt)
                    .add(+1, "days")
                    .format("YYYY-MM-DD")
            );
        },

        setToday() {
            let dt = this.$moment().format("YYYY-MM-DD");
            if (this.isRange) this.date_selected = [dt, dt];
            if (!this.isRange && !this.isWeek) this.date_selected = dt;
            this.changeDate(dt);
        },

        applyDate(val) {
            if (!this.isRange && !this.isWeek) this.date_selected = val;

            if (this.isWeek) {
                if (this.$moment(val).format("d") != 1) {
                    for (let i = 0; i < 7; i++) {
                        val = this.$moment(val)
                            .add(-1, "days")
                            .format("YYYY-MM-DD");
                        if (this.$moment(val).format("d") == 1) break;
                    }
                }
                this.date_selected = [
                    val,
                    this.$moment(val)
                        .add(6, "days")
                        .format("YYYY-MM-DD"),
                ];
            }
            if (this.isRange && this.date_selected.length < 2) {
                return;
            }

            if (this.date_selected.length < 9)  this.date_selected = this.date_selected + "-01";

            this.date_dialog = false;
        },

        changeDate(val) {
            if (val) this.applyDate(val);

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
