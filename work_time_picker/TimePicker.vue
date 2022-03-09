<template>
    <div class="ma-2" v-if="dates">
        <div class="header-hours d-flex">
            <div class="day-cell">день</div>
            <div class="hour-cell" v-for="(hour, hi) in hours" :key="hi">{{ hour }}ч</div>
        </div>

        <div class="week-day d-flex" v-for="(day, di) in days" :key="di">
            <div class="day-cell" :class="{ 'is-disabled': day.date < current_date }">
                {{ day.dt }} - <b>{{ weekdays[di] }}</b>
            </div>
            <div
                class="hour-cell is-selectable"
                :class="{ 'is-selected': hour.sel, 'is-disabled': day.date < current_date }"
                v-for="(hour, hi) in day.hours"
                :key="hi"
                :draggable="true"
                @dragstart="dragHandle('e-dragstart', $event, day, hour)"
                @dragover="dragHandle('e-dragover', $event, day, hour)"
                @dragend="dragHandle('e-dragend', $event, day, hour)"
                @click.stop="togleSelect(day, hour)"
            >
                {{ hour.hour }}
            </div>
        </div>

        <div class="d-flex">
            <v-btn @click="reset" class="mt-2" v-if="dates[1] >= current_date"> Очистить </v-btn>
            <v-spacer />
            <v-btn @click="save" class="mt-2" color="primary" v-if="dates[1] >= current_date"> Сохранить </v-btn>
        </div>
    </div>
</template>

<script>
let preX, preY, newStatus;

export default {
    props: {
        dates: { type: Array, default: null },
        selectedTimes: { type: Array, default: null },
    },
    watch: {
        selectedTimes() {
            this.refresh();
        },
        dates() {
            this.refresh();
        },
    },
    data() {
        return {
            //7days by 12hrs
            current_date: "2022-01-01",
            hours: [7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19],
            days: [],
            weekdays: ["Пн.", "Вт.", "Ср.", "Чт.", "Пт.", "Сб.", "Вс."],
        };
    },
    computed: {},
    mounted() {
        this.refresh();
    },
    methods: {
        refresh() {
            this.current_date = this.$moment().format("YYYY-MM-DD");
            let monday = this.dates[0];
            this.days = [];
            for (let i = 0; i < 7; i++) {
                this.days.push({
                    id: i,
                    dt: this.$moment(monday)
                        .add(i, "days")
                        .format("DD.MM"),
                    date: this.$moment(monday)
                        .add(i, "days")
                        .format("YYYY-MM-DD"),
                    hours: this.hours.map((e, eid) => {
                        return { id: eid, hour: e, sel: false };
                    }),
                });
            }

            if (this.selectedTimes && this.selectedTimes.length > 0) {
                this.days.forEach((day, di) => {
                    day.hours.forEach((hour, hi) => {
                        hour.sel = this.selectedTimes[di].hours[hi].sel;
                    });
                });
            }
        },
        togleSelect(day, hour) {
            if (this.days[day.id].date < this.current_date) return;
            this.days[day.id].hours[hour.id].sel = !this.days[day.id].hours[hour.id].sel;
            this.$emit("change", this.days);
        },

        dragHandle(type, $event, day, hour) {
            $event.stopPropagation();
            switch (type) {
                case "e-dragstart":
                    newStatus = !this.days[day.id].hours[hour.id].sel;
                    if (this.days[day.id].date < this.current_date) return;
                    this.days[day.id].hours[hour.id].sel = newStatus;
                    preX = day;
                    preY = hour;
                    break;
                case "e-dragover":
                    if (preX == day && preY == hour) return;
                    if (this.days[day.id].date < this.current_date) return;
                    this.days[day.id].hours[hour.id].sel = newStatus;
                    preX = day;
                    preY = hour;
                    break;
                case "e-dragend":
                    preX = null;
                    preY = null;
                    this.$emit("change", this.days);
                    break;
            }
        }, //dragHandle

        reset() {
            this.days.forEach((day) => {
                day.hours.forEach((hour) => {
                    hour.sel = false;
                });
            });
            this.$emit("reset-selected");
        },
        save() {
            this.$emit("save", this.days);
        },
    },
};
</script>

<style scoped>
.day-cell {
    text-align: center;
    border: solid 1px #ccc;
    width: 90px;
    height: 30px;
    line-height: 28px;
}
.hour-cell {
    text-align: center;
    border: solid 1px #ccc;
    width: 32px;
    height: 30px;
    line-height: 28px;
}
.header-hours {
    font-size: 14px;
    background: var(--v-timerbar-base);
}
.week-day {
    font-size: 13px;
}
.is-selectable {
    cursor: pointer;
}
.is-disabled {
    background: #f0f0f0;
    color: #a0a0a0;
}
.theme--light .is-selected {
    border: solid 1px #8dc5fc;
    background: #bbdcfc;
}
.theme--dark .is-selected {
    border: solid 1px #8dc5fc;
    background: #004080;
}
</style>
