<template>
    <section id="table">
        <div class="text-center" style="margin-top:30px;" v-if="!info">
            <v-progress-circular :size="70" :width="7" color="green" indeterminate></v-progress-circular>
        </div>
        <v-data-table
            v-if="info"
            multi-sort
            :headers="columnsFiltered"
            :fixed-header="fixedHeader"
            :items="rows"
            :loading="isLoading"
            loading-text="Загрузка..."
            class="elevation-4"
            :search="localTextFilter"
            :item-key="pagination.key"
            :items-per-page="pagination.itemsPerPage"
            :server-items-length="pagination.totalItems"
            :footer-props="{
                'items-per-page-options': info.itemsPerPageVariants,
                'show-current-page': true,
            }"
            :hide-default-header="hideDefaultHeader"
            :hide-default-footer="hideDefaultFooter"
            v-model="selected"
            :show-select="multiple"
            :single-select="!multiple"
            :show-expand="tableExpandable"
            :height="calcTableHeight"
            :mobile-breakpoint="0"
            @click:row="clickOnRow"
            @update:options="changeOptions"
        >

            <!-- TOP PANEL !-->
            <template v-slot:top v-if="showTop">
                <template v-if="$slots.top">
                    <slot name="top"></slot>
                </template>
                <v-toolbar v-else flat color="toolbar">
                    <v-btn icon @click="reloadTable()">
                        <v-icon>cached</v-icon>
                    </v-btn>

                    <v-divider inset vertical></v-divider>
                    <v-toolbar-title class="mx-2">{{ caption }}</v-toolbar-title>

                    <v-divider inset vertical></v-divider>
                    <v-btn color="primary" class="mx-2" @click="rowFilterDialog = true">
                        <v-icon>search</v-icon> Фильтр
                    </v-btn>

                    <v-divider inset vertical></v-divider>
                    <v-text-field
                        class="mx-3"
                        v-model="localTextFilter"
                        append-icon="search"
                        :label="pagination.totalItems > 0 ? 'Используйте ФИЛЬТР' : 'Поиск...'"
                        :disabled="pagination.totalItems > 0"
                        dense
                        outlined
                        hide-details
                        clearable
                        @click:clear.prevent="localTextFilter = ''"
                    ></v-text-field>

                    <v-divider inset vertical></v-divider>
                    <v-tooltip top color="indigo">
                        <template v-slot:activator="{ on }">
                            <v-btn v-on="on" icon @click="columnFilterDialog = true">
                                <v-icon>mdi-eye</v-icon>
                            </v-btn>
                        </template>
                        <span>Видимость колонок</span>
                    </v-tooltip>
                </v-toolbar>
            </template>
            <!-- END -- TOP PANEL !-->


            <!-- RENDER CELL ITEM !-->
            <template v-slot:item="{ item, select, isSelected, expand, isExpanded }">
                <tr :class="{
                        'v-data-table__selected': isSelected,
                        'v-data-table__expanded': isExpanded,
                    }"
                    @click="clickOnRow(item)"
                    @dblclick="dblClickOnRow(item)"
                >
                    <td width="30px" v-if="multiple">
                        <v-simple-checkbox :value="isSelected"
                            @input="
                                (val) => {
                                    select(val);
                                    clickOnRow(item, true, isSelected);
                                }
                            "
                            :ripple="false"
                        ></v-simple-checkbox>
                    </td>
                    <td width="30px" v-if="tableExpandable">
                        <v-btn icon
                            @click="
                                $emit('expand_item', item);
                                expand(!isExpanded);
                            "
                            ><v-icon>{{ `${isExpanded ? "expand_more" : "chevron_right"}` }}</v-icon></v-btn
                        >
                    </td>
                    <template v-for="columnItem in columnsFiltered">
                        <td :class="[{ 'text-center': columnItem.align }, columnItem.class]"
                            :key="columnItem.id"
                            v-if="!columnItem.hidden"
                            v-html="renderItem(columnItem, item, false)"
                        ></td>
                    </template>
                </tr>
            </template>

            <!-- RENDER EXPANDED ITEM !-->
            <template v-slot:expanded-item="{ headers,item }">
                <template v-if="$slots.expanded">
                    <td></td>
                    <td :colspan="columnsFiltered.length"><slot name="expanded"></slot></td>
                </template>
                <template v-if="!$slots.expanded">
                    <td></td>
                    <template v-for="(ex_table,ex_id) in info.childrenTables">
                        <td :key="ex_id" :colspan="columnsFiltered.length" class="left">
                            <table-editor :tableName="ex_table.table" :tableParent="[{ table: tableName, field: ex_table.field, value: item.id }]" :minHeight="200" :showTop="false"></table-editor>
                        </td>
                    </template>
                </template>
            </template>


            <!-- APPEND FOOTER PANEL !-->
            <template v-slot:body.append="{ headers }" v-if="appendRowData">
                <tr class="footer-row">
                    <template v-for="head_item in columnsFiltered">
                        <td class="footer-cell"
                            :key="head_item.id"
                            v-html="renderItem(head_item, appendRowData, true)"
                        ></td>
                    </template>
                </tr>
            </template>
            <!-- END APPEND FOOTER PANEL !-->


            <!-- BOTTOM PANEL !-->
            <template v-slot:footer v-if="showFooter">
                <v-divider />
                <div class="table-footer pa-2" v-if="$slots.footer">
                    <slot name="footer"></slot>
                </div>
                <div class="table-footer pa-2" v-else>
                    <template v-if="$slots.before_footer_buttons">
                        <slot name="before_footer_buttons"></slot>
                    </template>

                    <v-tooltip top color="green">
                        <template v-slot:activator="{ on }">
                            <v-btn
                                class="mx-2"
                                v-if="userHasRoles(info.add)"
                                fab
                                small
                                v-on="on"
                                @click="openEditDialog('add')"
                            >
                                <v-icon dark>add</v-icon>
                            </v-btn>
                        </template>
                        <span>Добавить запись</span>
                    </v-tooltip>
                    <v-tooltip top color="indigo">
                        <template v-slot:activator="{ on }">
                            <v-btn
                                class="mx-2"
                                v-if="userHasRoles(info.edit)"
                                fab
                                small
                                v-on="on"
                                :disabled="selected.length == 0"
                                @click="openEditDialog('edit')"
                            >
                                <v-icon dark>edit</v-icon>
                            </v-btn>
                        </template>
                        <span>Изменить запись</span>
                    </v-tooltip>
                    <v-tooltip top color="red">
                        <template v-slot:activator="{ on }">
                            <v-btn
                                class="mx-2"
                                v-if="userHasRoles(info.delete)"
                                fab
                                small
                                v-on="on"
                                :disabled="selected.length == 0"
                                @click="openEditDialog('delete')"
                            >
                                <v-icon dark>delete</v-icon>
                            </v-btn>
                        </template>
                        <span>Удалить запись</span>
                    </v-tooltip>

                    <template v-if="$slots.after_footer_buttons">
                        <slot name="after_footer_buttons"></slot>
                    </template>
                </div>
            </template>
            <!-- END -- BOTTOM PANEL !-->
        </v-data-table>

        <mdp-edit-dialog
            :visible="editDialog.visible"
            :action="editDialog.action"
            :tableName="editDialog.tableName"
            :tableInfo="info"
            :rowId="editDialog.rowId"
            :title="editDialog.title"
            :titleColor="editDialog.titleColor"
            :buttons="editDialog.buttons"
            @close="editDialog.visible = false"
            @save="saveEditDialog"
        >
        </mdp-edit-dialog>

        <mdp-column-filter v-model="localColumns" :visible="columnFilterDialog" @close="columnFilterDialog = false" />
        <mdp-row-filter
            v-model="tableFilter"
            :info="info"
            :visible="rowFilterDialog"
            :row="rowFilterValues"
            @close="rowFilterDialog = false"
            @change="rowFilterDialog = false"
        />
    </section>
</template>

<script>
import { mapActions } from "vuex";

export default {
    name:"table-editor",
    components: {
        "mdp-edit-dialog": () => import("./editDialog.vue"),
        "mdp-column-filter": () => import("./columnFilterDialog.vue"),
        "mdp-row-filter": () => import("./rowFilterDialog.vue"),
    },

    props: {
        customRenderItem: { type: Function, default: null },
        afterReloadTable: { type: Function, default: null },
        customEditDialog: { type: Function, default: null },
        tableName: { type: String, default: "" },
        tableCaption: { type: String, default: "" },
        tableFilter: { type: Array, default: null },
        tableParent: { type: Array, default: null },
        showTop: { type: Boolean, default: true },
        showFooter: { type: Boolean, default: true },
        showChildrens: { type: Boolean, default: true },
        appendRowData: { type: Object, default: null },
        subHeight: { type: Number, default: 0 },
        minHeight: { type: Number, default: 0 },
        maxHeight: { type: Number, default: 0 },
        multiple: { type: Boolean, default: false },
        value: { type: Array, default: null },
        fixedHeader: { type: Boolean, default: true },
        hideDefaultHeader: { type: Boolean, default: false },
        hideDefaultFooter: { type: Boolean, default: false },
    },

    data() {
        return {
            isLoading: false,
            localTextFilter: "",
            pagination: { page:1, sortBy:[], sortDesc:[], itemsPerPage:100, totalItems: -1 },

            tableExpandable: false,
            caption: "",
            info: null,
            localColumns: null,
            columns: null,

            selected: [],
            rows: [],

            editDialog: {
                visible: false,
                action: "",
                tableName: "",
                tableInfo: null,
                rowId: 0,
            },
            columnFilterDialog: false,
            rowFilterDialog: false,
            rowFilterValues: {},
        };
    },
    watch: {
        tableName() {
            this.rowFilterValues = {};
            this.reloadTable();
        },
        tableFilter() {
            this.reloadTable();
        },
        tableParent() {
            this.reloadTable();
        },
        value() {
            this.selectRows(this.value);
        },
    },

    computed: {
        columnsFiltered() {
            try {
                return this.localColumns.filter((obj) => {
                    return obj.hidden !== true;
                });
            } catch (e) {
                return this.localColumns;
            }
        },
        calcTableHeight() {
            let h = 315 + this.subHeight;
            if (!this.showTop) h -= 60;
            if (!this.showFooter) h -= 60;
            if (this.hideDefaultFooter) h -= 55;
            if (this.minHeight>0) return this.minHeight+"px";
            return "calc(100vh - " + h + "px)";
        },
    },

    mounted() {
        this.rowFilterValues = {};
        this.reloadTable();

        this.$on("openEditDialog", (action) => {
            this.openEditDialog(action);
        });
        this.$on("setTextFilter", (text) => {
            this.localTextFilter = text;
        });
        this.$on("setTableFilter", (filter) => {
            this.tableFilter = filter;
        });
        this.$on("reloadTable", () => {
            this.reloadTable();
        });

        document.addEventListener('keyup', this.onKeyUp);
    },
    beforeDestroy() {
        document.removeEventListener('keyup', this.onKeyUp);
    },

    methods: {
        userHasRoles(roles) {
            if (!roles) return false;
            var user_roles = this.$store.state.auth.user.role_id;
            if (typeof user_roles != "array") user_roles = [user_roles];
            for (let id in user_roles) {
                if (roles.indexOf(parseInt(user_roles[id])) != -1) {
                    return true;
                }
            }
            return false;
        },

        reloadTable() {
            this.isLoading = true;
            this.rows = [];

            if (this.tableFilter) {
                let newValues = {};
                this.tableFilter.forEach((e) => {
                    newValues[e.field] = e.value;
                });
                this.rowFilterValues = { ...this.rowFilterValues, ...newValues };
            }

            this.$api("table", this.tableName, "get", { page: this.pagination.page, limit: this.pagination.itemsPerPage, sortBy: this.pagination.sortBy, sortDesc: this.pagination.sortDesc, filter: this.tableFilter, parent: this.tableParent })
                .then((response) => {
                    this.isLoading = false;
                    if (this.afterReloadTable) response = this.afterReloadTable(response);
                    this.pagination = response.pagination;
                    this.rows = response.rows;
                    this.info = JSON.parse(JSON.stringify(response.info));
                    this.columns = this.convertColumns(this.info.columns);
                    this.localColumns = this.convertColumns(response.info.columns, true);
                    if (this.tableCaption == "") {
                        this.caption = response.info.name;
                    } else {
                        this.caption = this.tableCaption;
                    }
                    this.selectRows(this.value);

                    if (this.showChildrens && this.info.showChildrens && this.info.childrenTables) { 
                        this.tableExpandable = this.showChildrens; 
                    }
                })
                .catch((error) => {
                    this.isLoading = false;
                });
        }, //reloadTable

        convertColumns(columns, isTable) {
            isTable = isTable || false;
            var rez = [];
            for (let item in columns) {
                var obj = columns[item];
                obj.text = obj.label;
                obj.value = item;
                obj.name = item;
                if (this.tableParent) {
                    this.tableParent.forEach(e=>{
                        if (obj.name==e.field) obj.protected = true;
                    });
                }
                if (isTable && ["date", "dateTime", "checkBox", "timestamp", "select", "linkTable"].includes(obj.type)) {
                    obj.value = item + "_text";
                    obj.name = item + "_text";
                }
                rez.push(obj);
            }
            return rez;
        },

        onKeyUp(event){
            //console.log(event)
            if (this.editDialog.visible) return;
            if (this.columnFilterDialog) return;
            if (this.rowFilterDialog) return;

            if (event.key=="Insert") {
                this.openEditDialog('add');
            }
            if (event.key=="Enter") {
                if (this.selected.length > 0) this.openEditDialog('edit');
            }
            if (event.key=="Delete") {
                if (this.selected.length > 0) this.openEditDialog('delete');
            }
        },

        selectRows(ids) {
            if (ids) {
                this.selected = [];
                for (let item in this.rows) {
                    if (ids.indexOf(parseInt(this.rows[item].id)) >= 0) this.selected.push(this.rows[item]);
                }
            }
        },

        clickOnRow(row, isMultiple, rowSelected) {
            if (isMultiple) {
                if (rowSelected) {
                    this.selected = this.selected.filter((e) => e.id != row.id);
                } else {
                    this.selected.push(row);
                }
            } else {
                this.selected = [row];
            }
            this.$emit(
                "input",
                this.selected.map((e) => e.id)
            );
            this.$emit("select", this.selected);
        },
        dblClickOnRow(row) {
            if (this.multiple) {
                this.selected.push(row);
            } else {
                this.selected = [row];
            }
            this.$emit(
                "input",
                this.selected.map((e) => e.id)
            );
            this.$emit("dblSelect", this.selected);
        },
        changeOptions(pageOpts){
            if (this.pagination.page == pageOpts.page && this.pagination.itemsPerPage == pageOpts.itemsPerPage && this.pagination.totalItems < 0) return;
            if (this.pagination.page == pageOpts.page && this.pagination.itemsPerPage == pageOpts.itemsPerPage && pageOpts.sortBy.length == 0 && pageOpts.sortDesc.length == 0) return;
            this.pagination.page = pageOpts.page;
            this.pagination.sortBy = pageOpts.sortBy;
            this.pagination.sortDesc = pageOpts.sortDesc;
            this.pagination.itemsPerPage = pageOpts.itemsPerPage;
            this.reloadTable(pageOpts);
        },

        vifCalc(vif, row) {
            if (!vif || vif.length < 2) return true;
            let rules = vif;
            rules = rules.replace(/\[(.*?)\]/gi, (match, name) => {
                if (!row[name]) return 0;
                return row[name];
            });
            return eval(rules);
        },

        renderItem(head, item, isFooter) {
            if (item[head.name] == "undefined" || item[head.name] === null || item[head.name] === false) return "";
            let value = item[head.name];
            if (head.vif && !this.vifCalc(head.vif, item)) value = "";

            if (head.type == "password") value = "******";
            if (head.type == "images") {
                var images = "";
                for (let id in value) {
                    images += "<img src='" + value[id].src + "' height='48'> ";
                }
                value = images;
            }
            if (head.type == "files") {
                var files = "";
                for (let id in value) {
                    files += "<a href='" + value[id].url + "' >" + value[id].name + "</a> | ";
                }
                value = files;
                if (files.length > 0) value = " | " + files;
            }
            if (head.type == "float")
                value = parseFloat(value)
                    .toFixed(2)
                    .toString()
                    .replace(/\d(?=(\d{3})+\.)/g, "$& ");

            if (this.customRenderItem) {
                let newvalue = this.customRenderItem(head, item, isFooter);
                if (newvalue !== null) value = newvalue;
            }
            return value;
        },

        openEditDialog(action) {
            if (!this.customEditDialog) this.editDialog.visible = true;
            this.editDialog.action = action;
            this.editDialog.tableName = this.tableName;
            this.editDialog.tableInfo = this.info;
            this.editDialog.rowId = this.selected[0] ? this.selected[0].id : 0;
            this.editDialog.rowIndex = this.rows.indexOf(this.selected[0]);
            this.editDialog.row = this.selected[0] ? this.selected[0] : {};
            this.editDialog.titleColor = "primary";
            this.editDialog.title = "Добавление записи";
            this.editDialog.buttons = {
                left: { text: "Отмена", color: "red", icon: "close" },
                right: { text: "Сохранить", color: "primary", icon: "save" },
            };

            if (action == "add") {
                this.editDialog.rowId = 0;
                this.editDialog.rowIndex = 0;
                this.editDialog.row = {};
            }
            if (action == "edit") {
                this.editDialog.title = "Изменение записи";
            }
            if (action == "delete") {
                this.editDialog.titleColor = "red";
                this.editDialog.title = "Удаление записи";
                this.editDialog.buttons = {
                    left: { text: "Отмена", color: "primary", icon: "close" },
                    right: { text: "Удалить", color: "red", icon: "delete" },
                };
            }
            if (this.customEditDialog) this.customEditDialog( JSON.parse(JSON.stringify(this.editDialog)) );
        },
        saveEditDialog(row) {
            this.$swal.showLoading();

            if (this.tableParent) {
                this.tableParent.forEach((e) => {
                    if (!row[e.field]) row[e.field] = e.value;
                });
            }

            this.$api("table", this.editDialog.tableName, this.editDialog.action, this.editDialog.rowId, row)
                .then(async (response) => {
                    this.$swal.close();
                    if (response.error !== 0) { 
                        this.$swal.toastError(response.message, "center-center"); 
                        return; 
                    }
                    this.saveEditDialogFinish(response.rows[0]);
                    this.$emit("save", response.rows[0]);
                })
                .catch((error) => {
                    this.$swal.close();
                });
        },
        saveEditDialogFinish(row) {
            this.editDialog.visible = false;
            if (this.editDialog.action == "add") {
                this.rows.push(row);
                this.selected = [row];
            }
            if (this.editDialog.action == "edit") {
                this.rows[this.editDialog.rowIndex] = row;
            }
            if (this.editDialog.action == "delete") {
                this.rows.splice(this.editDialog.rowIndex, 1);
                this.selected = [];
            }
            this.rows = JSON.parse(JSON.stringify(this.rows));
            this.$forceUpdate();
        },
    }, //methods
};
</script>