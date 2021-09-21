<template>
    <section id="tree-table">
        <v-card class="mx-auto elevation-8">
            <v-card-title>
                <v-btn @click="openEditDialog({ id: 0 }, 'add')">Добавить пункт</v-btn>
                <v-spacer></v-spacer>
                <v-btn icon @click="reloadTable()">
                    <v-icon>cached</v-icon>
                </v-btn>
            </v-card-title>

            <v-divider></v-divider>
            <v-card-text>
                <el-tree
                    :data="menu_items"
                    node-key="id"
                    empty-text="Меню нет"
                    default-expand-all
                    draggable
                    @node-drop="changeTreeMenu"
                    :allow-drop="allowDrop"
                    :allow-drag="allowDrag"
                >
                    <template #default="{ node, data }">
                        <span class="custom-tree-node">
                            <div @dblclick="openEditDialog(data, 'edit')">
                                <span class="el-tree-node__label">{{ data.name }}</span>
                            </div>
                            <div class="mb-1">
                                <el-button
                                    icon="el-icon-plus"
                                    circle
                                    size="mini"
                                    type="success"
                                    @click.stop="openEditDialog(data, 'add')"
                                ></el-button>
                                <el-button
                                    icon="el-icon-edit"
                                    circle
                                    size="mini"
                                    type="primary"
                                    @click.stop="openEditDialog(data, 'edit')"
                                ></el-button>
                                <el-button
                                    icon="el-icon-delete"
                                    circle
                                    size="mini"
                                    type="danger"
                                    @click.stop="openEditDialog(data, 'delete')"
                                ></el-button>
                            </div>
                        </span>
                    </template>
                </el-tree>
            </v-card-text>
        </v-card>

        <!-- Диалог изменения пункта !-->
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
        <!-- ------------------------ !-->
    </section>
</template>

<script>
export default {
    components: {
        "mdp-edit-dialog": () => import("./editDialog.vue"),
    },

    props: {
        tableName: { type: String, default: "" },
        tableCaption: { type: String, default: "" },
    },

    data() {
        return {
            menu_type: 1,
            menu_items: [],
            info: null,
            columns: [],
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
            this.reloadTable();
        },
    },
    mounted() {
        this.reloadTable();
    },
    computed: {},

    methods: {
        showLoader(display) {
            this.$store.commit("SHOW_LOADER", display);
        },

        reloadTable() {
            if (!this.tableName) return;

            this.showLoader(true);
            this.$api("table", "tree", this.tableName)
                .then((response) => {
                    this.showLoader(false);
                    this.menu_items = response.rows;
                    this.info = response.info;
                    this.columns = response.info.columns;
                })
                .catch(() => {
                    this.showLoader(false);
                });
        },

        allowDrop(draggingNode, dropNode, type) {
            return true;
        },
        allowDrag(draggingNode) {
            return true;
        },

        findMaxId() {
            let maxId = 0;
            const cb = (e) => {
                if (e.id > maxId) maxId = e.id;
                if (e.children) e.children.forEach(cb);
            };
            this.menu_items.forEach(cb);
            return maxId + 1;
        },

        openEditDialog(item, action) {
            this.editDialog.action = action;
            this.editDialog.tableName = this.tableName;
            this.editDialog.tableInfo = this.info;
            this.editDialog.rowId = item? item.id : 0;
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
                this.editDialog.row = {};
                this.editDialog.parentId = item && item.id>0 ? item.id : 0;
                this.editDialog.row.name = "";
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
            this.editDialog.visible = true;
        },

        saveEditDialog(row) {
            if (this.editDialog.parentId) row.parent_id = this.editDialog.parentId;

            this.$swal.showLoading();
            this.$api("table", "tree/"+this.editDialog.tableName, this.editDialog.action, this.editDialog.rowId, row)
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
        saveEditDialogFinish(item) {
            this.editDialog.visible = false;

            if (this.editDialog.action == "add") {
                if (item.parent_id == 0) {
                    this.menu_items.push(item);
                } else {
                    const cb = (e, i, o) => {
                        if (e.id == item.parent_id) {
                            if (!e.children) e.children = [];
                            e.children.push(item);
                            return e;
                        }
                        if (e.children) e.children = e.children.map(cb);
                        return e;
                    };
                    this.menu_items = this.menu_items.map(cb);
                }
            }

            if (this.editDialog.action == "edit") {
                const cb = (e) => {
                    if (e.id == item.id) {
                        e = item;
                        return e;
                    }
                    if (e.children) e.children = e.children.map(cb);
                    return e;
                };
                this.menu_items = this.menu_items.map(cb);
            }

            if (this.editDialog.action == "delete") {
                const cb = (e) => {
                    if (e.id == item.id) return false;
                    if (e.children) e.children = e.children.filter(cb);
                    return true;
                };
                this.menu_items = this.menu_items.filter(cb);
            }
        },

        changeTreeMenu(current, parent){
            //console.log("change", current, parent);
            let row = {id: current.data.id, parent_id: parent.data.id, sort: current.level };
            this.$swal.showLoading();
            this.$api("table", "tree/"+this.tableName, "edit", current.data.id, row)
                .then(async (response) => {
                    this.$swal.close();
                })
                .catch((error) => {
                    this.$swal.close();
                });
        },

    }, //methods
};
</script>

<style>
#tree-table .custom-tree-node {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding-right: 4px;
    border-bottom: 1px dashed #ccc;
    font-size: 18px;
}

#tree-table .el-tree-node__label {
    padding: 0;
    font-size: 18px;
}

#tree-table .el-tree-node__content {
    height: 38px;
}
</style>
