<template>
    <v-dialog v-model="visible" scrollable persistent max-width="990px">
        <v-card v-if="visible">
            <v-card-title class="pa-0">
                <v-toolbar dense :color="titleColor" elevation="0">
                    <v-toolbar-title class="white--text">{{ title }}</v-toolbar-title>

                    <v-spacer></v-spacer>

                    <v-divider vertical dark class="mx-2" />
                    <v-btn icon dark dense small @click="downloadRowToFile()" class="mx-2" title="Скачать в виде файла">
                        <v-icon>mdi-download</v-icon>
                    </v-btn>
                    <v-btn icon dark dense small @click="$refs.fileSelector.$refs.input.click();" class="mx-2" title="Загрузить из файла">
                        <v-icon>mdi-upload</v-icon>
                    </v-btn>
                    <v-file-input style="display: none" ref="fileSelector" show-size v-model="selectedFile" @change="onChangeSelectedFile()" accept=".json"></v-file-input>
                    <v-divider vertical dark class="mx-2" />

                    <v-btn icon dark @click="$emit('close')">
                        <v-icon>close</v-icon>
                    </v-btn>
                </v-toolbar>
            </v-card-title>

            <v-card-title class="pa-0">
                <v-stepper ref="formStepper" non-linear flat v-if="pagination" style="width:100%; height:48px; background:var(--v-toolbar-base)" :value="page" @change="changePage">
                    <v-stepper-header>
                        <template v-for="n in (parseInt(pagination)-1)">
                            <v-stepper-step editable :step="n"></v-stepper-step>
                            <v-divider></v-divider>
                        </template>
                        <v-stepper-step editable :step="pagination"></v-stepper-step>
                    </v-stepper-header>
                </v-stepper>
            </v-card-title>

            <v-card-text class="pa-4" v-if="action=='delete'">
                Вы точно хотите удалить эту запись?
                <v-card-actions class="mt-4 pa-0">
                <v-btn :color="buttons.left.color" @click="$emit('close')">
                    <v-icon left>{{ buttons.left.icon }}</v-icon> {{ buttons.left.text }}
                </v-btn>
                <v-spacer></v-spacer>
                <v-btn :color="buttons.right.color" @click="$emit('save', {id:rowId})">
                    <v-icon left>{{ buttons.right.icon }}</v-icon> {{ buttons.right.text }}
                </v-btn>
                </v-card-actions>
            </v-card-text>

            <v-card-text class="pa-0" v-else>
                <mdp-edit-form
                    v-if="localTableInfo"
                    ref="mdpEditForm"
                    :action="action"
                    :tableName="tableName"
                    :tableInfo="localTableInfo"
                    :rowId="rowId"
                    :row="localRow"
                    :buttons="buttons"
                    :page="page"
                    :reloadRow="reloadRow"
                    @close="$emit('close')"
                    @change="can_save = true"
                    @setpagination="pagination = $event; page = 1;"
                    :visible="visible"
                    @save="save"
                >
                </mdp-edit-form>
            </v-card-text>


            <v-divider></v-divider>
            <v-card-actions v-if="action!='delete'" style="background:var(--v-toolbar-base)">
                <v-btn v-if="pagination && page>1" class="mx-2" color="primary" outlined @click="prevPage">
                    <v-icon left>mdi-chevron-double-left</v-icon> Назад
                </v-btn>
                <v-btn v-else class="mx-2" :color="buttons.left.color"  @click="$emit('close')">
                    <v-icon left>{{ buttons.left.icon }}</v-icon> {{ buttons.left.text }}
                </v-btn>
                <v-spacer></v-spacer>
                <v-btn v-if="pagination && page<pagination" class="mx-2" color="primary" outlined @click="nextPage">
                    Далее <v-icon right>mdi-chevron-double-right</v-icon>
                </v-btn>
                <template v-else>
                    <v-btn v-if="action != 'view'" class="mx-2" :color="buttons.right.color"  :disabled="!can_save" @click="$refs.mdpEditForm.save()">
                        <v-icon left>{{ buttons.right.icon }}</v-icon> {{ buttons.right.text }}
                    </v-btn>
                </template>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script>
import { mapActions } from "vuex";

export default {
    components: {
        "mdp-edit-form": () => import("./editForm.vue"),
    },

    props: {
        visible: { type: Boolean, default: false },
        titleColor: { type: String, default: "primary" },
        title: { type: String, default: "Редактирование записи" },
        action: { type: String, default: "add" },
        tableName: { type: String, default: "" },
        tableInfo: { type: Object, default: null },
        rowId: { type: Number, default: 0 },
        row: { type: Object, default: null },
        buttons: {
            type: Object,
            default: () => {
                return {
                    left: { text: "Отмена", color: "red", icon: "close" },
                    right: { text: "Сохранить", color: "primary", icon: "save" },
                };
            },
        },
    },

    data() {
        return {
            selectedFile: null,
            selectedFileData: null,
            pagination: false,
            page: 1,
            can_save: false,
            localTableInfo: null,
            localRow: {},
            reloadRow: true,
        };
    },

    watch: {
        visible() {
            if (this.visible) this.onShowDialog();
        },
    },

    mounted() {
    },
    beforeDestroy() {
    },

    methods: {
        onShowDialog() {
            this.reloadRow = true;
            this.localTableInfo = null;
            this.localRow = {};

            if (this.row && this.tableInfo) {
                this.localTableInfo = JSON.parse(JSON.stringify(this.tableInfo));
                this.localRow = JSON.parse(JSON.stringify(this.row));
                return;
            }

            this.$api("table", this.tableName, this.rowId)
                .then((response) => {
                    if (response.rows[0]) this.localRow = response.rows[0];
                    this.localTableInfo = response.info;
                    this.reloadRow = false;
                })
                .catch((error) => {
                });
        },

        onChangeSelectedFile() {
            this.localRow = {};
            this.selectedFileData = {
                name: this.selectedFile.name,
                size: this.selectedFile.size,
                src: "",
            };
            var reader = new FileReader();
            reader.onload = (e) => {
                this.selectedFileData.src = e.target.result;
                let fileSrc = decodeURIComponent(escape(window.atob(this.selectedFileData.src.substr(29))));
                let json = JSON.parse(fileSrc);
                this.localRow = json;
                this.selectedFile = null;
            };
            reader.readAsDataURL(this.selectedFile);
        },
        downloadRowToFile() {
            const contentType = "application/octet-stream";
            const link = document.createElement("a");
            link.href = window.URL.createObjectURL(new Blob([JSON.stringify(this.localRow)], { type: contentType }));
            link.setAttribute("download", this.rowId + "-" + this.tableName + ".json");
            document.body.appendChild(link);
            link.click();
            link.remove();
        },


        prevPage(){
            if (this.page <= 1) return;
            this.page = this.page - 1;
        },
        nextPage(){
            if (!this.$refs.mdpEditForm.isValid()) { this.$nextTick(()=>{ this.$refs.formStepper.internalValue = this.page; }); this.$swal.toastError("Заполните все поля правильно", "center-center", 1500); return; }
            this.page = this.page + 1;
        },
        changePage(newPage){
            if (!this.$refs.mdpEditForm.isValid()) { this.$swal.toastError("Заполните все поля правильно", "center-center", 1500); return; }
            this.page = newPage;
        },

        save(row) {
            this.$emit("save", row);
        },
    },
};
</script>

<style>
.v-stepper__header {
    height: 48px !important;
}
.v-stepper__step {
   padding: 2px 24px !important;
}
</style>
