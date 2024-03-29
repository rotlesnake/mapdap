<template>
    <section id="tree-table">
        <v-card class="mx-auto elevation-8" v-if="info">
            <slot name="header">
            <v-card-title>
                {{info.name}}
                <v-spacer></v-spacer>
                <v-btn color="primary" @click="openEditDialog({ id: 0 }, 'add')">Добавить пункт</v-btn>
                <v-spacer></v-spacer>
                <v-btn class="mr-2" @click="openAll" fab small><v-icon>mdi-expand-all</v-icon></v-btn>
                <v-btn class="mr-6" @click="closeAll" fab small><v-icon>mdi-collapse-all</v-icon></v-btn>

                <v-btn fab small @click="reloadTable()" color="primary">
                    <v-icon>cached</v-icon>
                </v-btn>
            </v-card-title>
            <v-divider></v-divider>
            </slot>

            <v-card-text>
                <el-tree
                    :data="menu_items"
                    node-key="id"
                    :empty-text="emptyText"
                    :default-expanded-keys="opened"
                    :draggable="draggable"
                    @node-drop="changeTreeMenu"
                    @node-expand="opened.push($event.id)"
                    @node-collapse="collapseNode"
                    :allow-drop="allowDrop"
                    :allow-drag="allowDrag"
                >
                    <template #default="{ node, data }">
                    <slot v-bind:item="data">
                        <span class="custom-tree-node">
                            <div @dblclick="openEditDialog(data, 'edit')">
                                <slot name="item" v-bind:item="data">
                                    <span class="el-tree-node__label">{{ data.name }}</span>
                                </slot>
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
                    </slot>
                    </template>
                </el-tree>
            </v-card-text>

            <slot name="footer">
            </slot>
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
        draggable: { type: Boolean, default: true },
        tableName: { type: String, default: "" },
        tableCaption: { type: String, default: "" },
        emptyText: {type: String, default:"Нет данных"},
        tableFilter: { type: Object, default: null },
        afterReloadTable: { type: Function, default: null },
        customEditDialog: { type: Function, default: null },

        tableParent: { type: Array, default: null },
    },

    data() {
        return {
            menu_type: 1,
            menu_items: [],
            info: null,
            columns: [],
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
            this.reloadTable();
        },
        tableFilter() {
            this.reloadTable();
        },
        tableParent(oldval,newval) {
            if (oldval.field == newval.field && oldval.value == newval.value) return;
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
            let filter = {};
            if (this.tableFilter) filter = this.tableFilter;

            this.$api("table", "tree", this.tableName, "get", { parent: this.tableParent, ...filter })
                .then((response) => {
                    if (this.afterReloadTable) response = this.afterReloadTable(response);
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
                this.customEditDialog( JSON.parse(JSON.stringify(this.editDialog)) );
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
            if (this.editDialog.action=="copy") this.editDialog.action="add";
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
                        let child=null; 
                        if (e.children) child = e.children.map(cb);
                        e = item;
                        e.children = child;
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

        changeTreeMenu(current, parent, position){
            let row = {id: current.data.id, parent_id: parent.data.id, sort: 0 };
            if (position == "before") {row.parent_id = parent.data.parent_id; row.sort = parent.data.sort-1; }
            if (position == "after")  {row.parent_id = parent.data.parent_id; row.sort = parent.data.sort+1; }
            current.data.parent_id = row.parent_id;
            current.data.sort = row.sort;

            if (this.tableParent) {
                this.tableParent.forEach((e) => {
                    if (!row[e.field]) row[e.field] = e.value;
                });
            }

            this.$swal.showLoading();
            this.$api("table", "tree/"+this.tableName, "edit", current.data.id, row)
                .then(async (response) => {
                    //current.data.sort = response.rows[0]["sort"];
                    //console.log(response.rows[0]["sort"])
                    this.reloadTable();
                    this.$swal.close();
                })
                .catch((error) => {
                    this.$swal.close();
                });
        },

        openAll() {
            this.opened = [];
            this.menu_items.forEach((e) => {
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
            let arhiv = this.menu_items;
            this.menu_items = [];
            setTimeout(() => {
                this.menu_items = arhiv;
            }, 10);
        },
        collapseNode(node) {
            this.opened = this.opened.filter(e=>e!=node.id);
        }
    }, //methods
};
</script>

<style>
.theme--dark .el-tree {
  background: var(--v-background-lighten2);
  color:#fff;
}
.theme--dark .el-tree-node__content:hover, .theme--dark .el-tree-node:focus .el-tree-node__content {
  background: var(--v-background-lighten3);
}
.theme--dark .is-current {
  background: var(--v-background-lighten3);
}

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
