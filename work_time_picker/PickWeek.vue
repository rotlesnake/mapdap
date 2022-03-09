<template>
    <div>
        <v-dialog v-model="date_dialog" width="300px">
            <v-date-picker
                v-model="date_range"
                locale="ru-RU"
                :first-day-of-week="1"
                :weekday-format="weekdayFormat"
                @click:date="changeDate"
                range
                scrollable
            ></v-date-picker>
        </v-dialog>
        <v-row justify="center">
            <div class="title text-center">
                <div class="my-1">Рабочая неделя</div>
                <div class="d-flex">
                    <v-btn class="" @click="prevWeek"><<<</v-btn>
                    <v-card outlined class="mx-4 px-1 elevation-1" @click="date_dialog = true">
                        <v-icon color="indigo">mdi-calendar-clock</v-icon>
                        {{ dataRange }}
                    </v-card>
                    <v-btn class="" @click="nextWeek">>>></v-btn>
                </div>
            </div>
        </v-row>
    </div>
</template>

<script>
export default {
    components: {},
    data() {
        return {
            date_dialog: false,
            date_range: ["2022-01-01", "2022-01-01"],
            weekdays: ["Вс", "Пн", "Вт", "Ср", "Чт", "Пт", "Сб"],
        };
    },
    computed: {
        dataRange() {
            return this.$moment(this.date_range[0]).format("DD.MM.YYYY") + " - " + this.$moment(this.date_range[1]).format("DD.MM.YYYY");
        },
    },

    mounted() {
        this.changeDate(this.$moment().format("YYYY-MM-DD"));
    },

    methods: {
        weekdayFormat(val) {
            return this.weekdays[this.$moment(val).format("d")];
        },

        prevWeek() {
            const dt = this.date_range[0];
            this.changeDate(
                this.$moment(dt)
                    .add(-7, "days")
                    .format("YYYY-MM-DD")
            );
        },
        nextWeek() {
            const dt = this.date_range[0];
            this.changeDate(
                this.$moment(dt)
                    .add(+7, "days")
                    .format("YYYY-MM-DD")
            );
        },

        changeDate(val) {
            if (this.$moment(val).format("d") != 1)
                for (let i = 0; i < 7; i++) {
                    val = this.$moment(val)
                        .add(-1, "days")
                        .format("YYYY-MM-DD");
                    if (this.$moment(val).format("d") == 1) break;
                }
            this.date_range = [
                val,
                this.$moment(val)
                    .add(6, "days")
                    .format("YYYY-MM-DD"),
            ];
            this.date_dialog = false;

            this.$emit("change", this.date_range);
        },
    }, //methods
};
</script>

<style></style>
