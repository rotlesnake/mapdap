<template>
    <div class="ma-2 container">
        <div v-for="(day, di) in selectedTimes" class="row" :key="di">
            <span class="left" v-html="showDate(day, di)"></span>
            <span class="wrap">
                <span v-for="(section, sIndex) in timeSectionFormat(day.hours)" class="section-time" :key="sIndex">
                    {{ section }}
                </span>
            </span>
        </div>
    </div>
</template>

<script>

export default {
    name: "time-section",
    props: {
        dates: { type: Array, default: null },
        selectedTimes: { type: Array, default: null },
    },

    data() {
        return {
            weekdays: ["Пн.", "Вт.", "Ср.", "Чт.", "Пт.", "Сб.", "Вс."],
        };
    },
    computed: {},
    methods: {
        showDate(day, di) {
            return day.dt + " - <b>" + this.weekdays[di] + "</b>";
        },
        timeSectionFormat(hours) {
            let times = [false];
            let group = "";
            hours.forEach((e, i) => {
                if (!e.sel) {
                    if (times[times.length - 1] !== false) {
                        times.push(false);
                        group += hours[i - 1].hour + ";";
                    }
                    return;
                }
                if (times[times.length - 1] === false) {
                    group += e.hour + "-";
                }
                times.push(e.hour);
                if (i == hours.length - 1) group += e.hour;
            });
            if (times.length < 2) return [];

            if (group.substr(-1) == ";") group = group.substr(0, group.length - 1);
            group = group.split(";").map((e) => {
                let obj = e.split("-");
                if (!obj[1]) return "";
                return obj[0] + ":00 ~ " + (parseInt(obj[1]) + 1) + ":00";
            });
            return group;
        },

        close() {
            this.$emit("close-time-section");
        },
    }, //methods
};
</script>

<style scoped>
.container {
    width: 520px;
    background: var(--v-primary);
    border: 1px solid #ccc;
    border-radius: 4px;
}

.row {
    font-size: 12px;
    min-height: 36px;
    line-height: 36px;
    border-bottom: #e6ebf5 solid 1px;
}

.row .left {
    padding: 0 0 0 10px;
    width: 100px;
}

.row .wrap {
    flex-flow: wrap;
}

.row .section-time {
    display: inline-block;
    width: 100px;
}
</style>
