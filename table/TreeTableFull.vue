<template>
    <section class="" v-if="columns">
        <div class="tree-table" v-if="columns">
            <div class="table-header">
                <div v-for="(col,ci) in activeColumns" :key="ci" class="header-column" :style="{ ...col.style, ...colStyle(col) }">
                     <div v-if="col.name == 'name' || col.name == 'field_label'" style="min-width: 48px; display:inline-block;">
                         <v-btn class="mr-1" @click="openAll" icon x-small><v-icon>mdi-expand-all</v-icon></v-btn>
                         <v-btn class="mr-2" @click="closeAll" icon x-small><v-icon>mdi-collapse-all</v-icon></v-btn>
                     </div>
                     {{ col.label }}
                </div>
                <div class="header-column" style="min-width: 120px; max-width: 120px">Действия <el-button icon="mdi mdi-plus" circle size="mini" type="success" @click.stop="openEditDialog({ id: 0, type: 1 }, 'add')"></el-button></div>
            </div>
            <el-tree
                :data="rows"
                node-key="id"
                empty-text="Нет данных"
                :default-expanded-keys="opened"
                @node-expand="opened.push($event.id)"
                @node-collapse="collapseNode"
                :draggable="draggable"
                @node-drop="changeTreeMenu"
                :allow-drop="allowDrop"
                :allow-drag="allowDrag"
            >
                <template #default="{ data }">
                    <div class="table-row" @dblclick="openEditDialog(data, 'edit')">
                        <div v-for="(col,ci) in activeColumns" :key="ci" class="row-column" :style="{ ...col.style, ...colStyle(col, data) }">
                            {{ data[col.name + "_text"]==undefined ? data[col.name] : data[col.name + "_text"] }}
                        </div>

                        <div class="ml-1">
                            <el-button icon="mdi mdi-briefcase-plus" circle size="mini" type="success" @click.stop="openEditDialog({ id: data.id, type: 1 }, 'add')"></el-button>
                            <el-button icon="mdi mdi-note-plus-outline" circle size="mini" type="success" @click.stop="openEditDialog({ id: data.id, type: 2 }, 'add')"></el-button>
                            <el-button icon="el-icon-edit" circle size="mini" type="primary" @click.stop="openEditDialog(data, 'edit')"></el-button>
                            <el-button icon="el-icon-delete" circle size="mini" type="danger" @click.stop="openEditDialog(data, 'delete')"></el-button>
                        </div>
                    </div>
                </template>
            </el-tree>
        </div>

        <!-- Диалог изменения пункта !-->
        <mdp-edit-dialog
            :visible="editDialog.visible"
            :action="editDialog.action"
            :tableName="editDialog.tableName"
            :tableInfo="info"
            :rowId="editDialog.rowId"
            :row="editDialog.row"
            :title="editDialog.title"
            :titleColor="editDialog.titleColor"
            :buttons="editDialog.buttons"
            @close="editDialog.visible = false"
            @save="saveEditDialog"
        >
        </mdp-edit-dialog>
        <!-- ------------------------ !-->
    </section>
</template>

<script>
export default {
    components: {
        "mdp-edit-dialog": () => import("./editDialog.vue"),
    },
    props: {
        draggable: { type: Boolean, default: true },
        drag_strict: { type: Boolean, default: false },
        tableName: { type: String, default: "" },
        tableCaption: { type: String, default: "" },
        tableFilter: { type: Array, default: null },
        afterReloadTable: { type: Function, default: null },
        customEditDialog: { type: Function, default: null },
        iconFolder: { type: String, default: "mdi-folder-outline" },
        iconElement: { type: String, default: "mdi-text-box-outline" },
        tableParent: { type: Array, default: null },
        folderBackground: { type: String, default: "#eee" },
    },

    data() {
        return {
            rows: null,
            info: null,
            columns: null,
            opened: [],
            editDialog: {
                visible: false,
                action: "",
                tableName: "",
                tableInfo: null,
                rowId: 0,
                parentId: 0,
                sort: 0,
            },
        };
    },
    watch: {
        tableName() {
            this.loadTreeTable();
        },
        tableFilter() {
            this.loadTreeTable();
        },
        tableParent(oldval, newval) {
            if (oldval.field == newval.field && oldval.value == newval.value) return;
            this.loadTreeTable();
        },
    },
    mounted() {
        this.loadTreeTable();
    },
    computed: {
        activeColumns() {
            return this.columns.filter((e) => !e.hidden && !e.hiddenTree);
        },
    },

    methods: {
        showLoader(display) {
            this.$store.commit("SHOW_LOADER", display);
        },

        loadTreeTable() {
            if (!this.tableName) return;
            this.showLoader(true);
            this.$api("table", "tree", this.tableName, "get", {tableParent: this.tableParent})
                .then((response) => {
                    this.showLoader(false);
                    this.rows = response.rows;
                    this.info = response.info;
                    this.columns = Object.values(response.info.columns);
                    this.$forceUpdate();
                })
                .catch(() => {
                    this.showLoader(false);
                });
        },

        colStyle(col, data) {
            let minwidth = 100;
            if (col.name == "name" || col.name == "field_label") minwidth = 250;
            if (col.style && col.style.width) minwidth = col.style.width;
            let style = { "min-width": minwidth + "px" };

            if (data && (col.name == "name" || col.name == "field_label")) {
                minwidth = minwidth - (data.tree_level + 1) * 18;
                style["min-width"] = minwidth + "px";
            }

            if (data && data.type == 1) { style["background"] = this.folderBackground;  style["border-bottom"] = "1px dashed #ccc"; style["font-weight"] = "600"; }
            if (data && (col.name == "name" || col.name == "field_label")) { style["text-align"] = "left"; } else { style["text-align"] = "center"; }

            style["max-width"] = style["min-width"];
            return style;
        },

        collapseNode(node) {
            this.opened = this.opened.filter((e) => e != node.id);
        },

        allowDrop(draggingNode, dropNode, type) {
            if (this.drag_strict && type == "inner" && draggingNode.data.type == 1 && dropNode.data.type == 2) return false;
            if (this.drag_strict && type == "inner" && draggingNode.data.type == 2 && dropNode.data.type == 2) return false;
            return true;
        },
        allowDrag(draggingNode) {
            return true;
        },

        openAll() {
            this.opened = [];
            this.rows.forEach((e) => {
                this.opened.push(e.id);
                if (e.children && e.children.length > 0) {
                    e.children.forEach((ec) => {
                        this.opened.push(ec.id);
                    });
                }
            });
        },
        closeAll() {
            this.opened = [];
            let arhiv = this.rows;
            this.rows = [];
            setTimeout(() => {
                this.rows = arhiv;
            }, 10);
        },

        openEditDialog(item, action) {
            this.editDialog.action = action;
            this.editDialog.tableName = this.tableName;
            this.editDialog.tableInfo = this.info;
            this.editDialog.rowId = item ? item.id : 0;
            this.editDialog.row = item || {};
            this.editDialog.parentId = null;
            this.editDialog.titleColor = "primary";
            this.editDialog.title = "Добавление записи";
            this.editDialog.buttons = {
                left: { text: "Отмена", color: "red", icon: "close" },
                right: { text: "Сохранить", color: "primary", icon: "save" },
            };

            if (action == "add") {
                this.editDialog.rowId = 0;
                this.editDialog.rowIndex = 0;
                this.editDialog.parentId = item && item.id > 0 ? item.id : 0;
                this.editDialog.row.id = 0;
            }
            if (action == "copy") {
                this.editDialog.rowId = 0;
                this.editDialog.rowIndex = 0;
                this.editDialog.title = "Дублирование записи";
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

            if (this.customEditDialog) {
                this.customEditDialog(JSON.parse(JSON.stringify(this.editDialog)));
            } else {
                this.editDialog.visible = true;
            }
        },

        saveEditDialog(row) {
            if (this.editDialog.parentId) row.parent_id = this.editDialog.parentId;

            if (this.tableParent) {
                this.tableParent.forEach((e) => {
                    if (!row[e.field]) row[e.field] = e.value;
                });
            }

            this.$swal.showLoading();
            if (this.editDialog.action == "copy") this.editDialog.action = "add";

                this.$api("table", "tree/" + this.editDialog.tableName, this.editDialog.action, this.editDialog.rowId, row)
                    .then(async (response) => {
                        this.$swal.close();
                        if (response.error !== 0) {
                            this.$swal.toastError(response.message, "center-center");
                            return;
                        }
                        this.$emit("save", response.rows[0]);
                        this.loadTreeTable();
                        this.editDialog.visible = false;
                    })
                    .catch((error) => {
                        this.$swal.close();
                    });
        },

        changeTreeMenu(current, parent, position) {
            let row = {
                id: current.data.id,
                parent_id: parent.data.id,
                sort: 0,
            };
            if (position == "before") {
                row.parent_id = parent.data.parent_id;
                row.sort = parent.data.sort - 1;
            }
            if (position == "after") {
                row.parent_id = parent.data.parent_id;
                row.sort = parent.data.sort + 1;
            }

            if (this.tableParent) {
                this.tableParent.forEach((e) => {
                    if (!row[e.field]) row[e.field] = e.value;
                });
            }

            this.$swal.showLoading();
            this.$api("table", "tree/" + this.tableName, "edit", current.data.id, row)
                .then(async (response) => {
                    this.$swal.close();
                    this.loadTreeTable();
                })
                .catch((error) => {
                    this.$swal.close();
                });
        },
    }, //methods
};
</script>

<style>
button.el-button {
    margin: 0 !important;
}
button.el-button i::before {
    font-size: 16px;
}
button.el-button--mini.is-circle {
    padding: 3px;
}

.tree-table {
    overflow: auto;
}
.tree-table .table-header {
    display: flex;
    width: fit-content;
}
.tree-table .table-header .header-column {
    width: 100%;
    min-width: 100px;
    border: 1px solid #999;
    text-align: center;
}

.tree-table .el-tree-node__children {
    width: fit-content !important;
}
.tree-table .table-row {
    display: flex;
    width: fit-content;
}
.tree-table .table-row .row-column {
    width: 100%;
    min-width: 100px;
    padding: 1px 4px;
    overflow: clip;
}
</style>
