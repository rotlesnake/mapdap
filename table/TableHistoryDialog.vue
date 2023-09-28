<template>
    <v-dialog v-model="visible" scrollable persistent>
        <v-card v-if="visible">
            <v-card-title class="pa-0">
                <v-toolbar dense color="primary" elevation="0">
                    <v-toolbar-title class="white--text">{{ title }}</v-toolbar-title>

                    <v-spacer></v-spacer>

                    <v-btn icon dark @click="close()">
                        <v-icon>close</v-icon>
                    </v-btn>
                </v-toolbar>
            </v-card-title>

            <v-card-text class="pa-4">
                <table style="width: 100%" cellpadding="0" cellspacing="0" border="1">
                    <tr>
                        <th class="">Дата и время изменения</th>
                        <th class="">Автор изменений</th>
                        <th class="">Событие</th>
                        <th class="">Описание изменений</th>
                    </tr>
                    <tr v-for="(row, i) in rows" :key="i">
                        <td class="" style="text-align:center">{{ row.created_at_text }}</td>
                        <td class="" style="text-align:center">{{ row.user_id }}.{{ row.user_id_text }}</td>
                        <td class="" style="text-align:center">{{ actionText[row.action] }}</td>
                        <td class="pa-2" v-html="row.desc">
                        </td>
                    </tr>
                </table>
            </v-card-text>
        </v-card>
    </v-dialog>
</template>

<script>
import { mapActions } from "vuex";

export default {
    components: {
    },

    props: {
    },

    data() {
        return {
            visible: false,
            title: "История",
            actionText:[
                "",
                "Добавлено",
                "Изменено",
                "Удалено",
            ],
            rows: [],
        };
    },

    mounted() {
    },
    beforeDestroy() {
    },

    methods: {
        show(tableName, row) {
            this.visible = true;
            this.title = "История изменения записи №"+row.id;
            this.rows = [];
            this.$api("system","logs","sys-log", {table:tableName, row_id:row.id}).then(data=>{
                this.rows = data.map(row=>{
                    row.desc = "<table width='100%' cellpadding='0' cellspacing='0' border='1'>";
                    row.desc += "<tr>";
                    row.desc += "  <th>Поле</th>";
                    row.desc += "  <th>Было</th>";
                    row.desc += "  <th>Стало</th>";
                    row.desc += "</tr>";
                    for(let key in row.fields) {
                        if (row.fields_pre[key] != row.fields[key]) {
                            if (String(row.fields[key]).substr(0,2)=="[{") {
                                row.desc += this.renderSubTable(key, JSON.parse(row.fields_pre[key] || "[]"), JSON.parse(row.fields[key] || "[]"));
                                continue;
                            }
                            row.desc += "<tr>";
                            row.desc += "  <td><b>"+key+"</b></td>";
                            row.desc += "  <td>"+(row.fields_pre[key] || "")+"</td>";
                            row.desc += "  <td>"+(row.fields[key])+"</td>";
                            row.desc += "</tr>";
                        }
                    }
                    row.desc += "</table>";
                    return row;
                });
            })
        },

        renderJsonTable(tbl, oldTbl) {
            let txt = "<table width='100%' cellpadding='0' cellspacing='0' border='1'>";
            for(let i in tbl) {
                let row = tbl[i];
                txt += "<tr><td>";
                for(let fld in row) {
                    if (Array.isArray(row[fld])) continue;
                    if (typeof row[fld] == "object") continue;
                    txt += "<b>"+fld+":</b> ";
                    if (oldTbl && oldTbl[i] && row[fld]!=oldTbl[i][fld]) {
                        txt += "<font color=red>"+row[fld]+"</font>";
                    } else {
                        txt += row[fld];
                    }
                    txt += "<br>";
                }
                txt += "</td></tr>";
            }
            txt += "</table>";
            return txt;
        },
        renderSubTable(field, oldVal, newVal) {
            let txt = "";
            let subTable = "";
            txt += "<tr>";
            txt += "  <td><b>"+field+"</b></td>";
            txt += "  <td>"+this.renderJsonTable(oldVal)+"</td>";
            txt += "  <td>"+this.renderJsonTable(newVal,oldVal)+"</td>";
            txt += "</tr>";
            return txt;
        },

        close() {
            this.visible = false;
        },
    },
};
</script>

