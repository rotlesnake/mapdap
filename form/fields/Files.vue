<template>
    <div id="field-files" class="mb-4">
        <v-card>
            <v-card-title class="my-2 body-1">
                {{ label }}
            </v-card-title>

            <v-card-title class="py-0 mt-n3" v-for="(item, i) in filesList" :key="i">
                <v-switch
                    dense
                    color="gray"
                    v-model="item.type"
                    :true-value="2"
                    :false-value="1"
                    :label="item.type == 1 ? 'файл' : ' URL '"
                ></v-switch>
                <v-btn v-if="multiple" class="ml-1" x-small fab color="red" dark @click.stop="deleteItem(i)"
                    ><v-icon>close</v-icon></v-btn
                >

                <template v-if="item.type == 1">
                    <v-file-input
                        class="ml-1"
                        chips
                        label="Файл"
                        v-model="file"
                        show-size
                        :rules="[
                            (v) =>
                                !v ||
                                (v && !v.size) ||
                                (v && v.size < 2 * 1024 * 1024) ||
                                'Размер файла должен быть не более 2 MB',
                        ]"
                        prepend-icon=""
                        @change="onFileLoad(i)"
                        hide-details
                        outlined
                        dense
                    ></v-file-input>
                </template>
                <template v-else>
                    <v-text-field
                        class="ml-1"
                        v-model="item.src"
                        label="URL файла"
                        hide-details
                        outlined
                        dense
                        @input="updateItem()"
                    ></v-text-field>
                </template>

                <input
                    v-if="item.src.length > 9"
                    class="comment"
                    placeholder="Комментарий"
                    v-model="item.caption"
                    @input="updateItem()"
                />
            </v-card-title>

            <v-divider v-if="multiple" />
            <v-card-actions v-if="multiple" class="py-0">
                <v-spacer />
                <v-btn fab small color="green" class="ma-1" dark @click.stop="addFileInList"
                    ><v-icon>add</v-icon></v-btn
                >
            </v-card-actions>
        </v-card>
    </div>
</template>

<script>
export default {
    props: {
        label: { type: String, default: null },
        value: { required: true },
        multiple: { type: Boolean, default: false },
    },
    data() {
        return {
            filesList: [],
            file: [],
        };
    },
    watch: {
        value() {
            this.refresh();
        },
    },
    mounted() {
        this.refresh();
    },

    methods: {
        refresh() {
            if (typeof this.value == "string") {
                try {
                    this.filesList = JSON.parse(this.value);
                } catch (e) {}
            } else {
                this.filesList = this.value || [];
                if (!this.value) this.addFileInList();
            }
            if (!this.multiple && this.filesList.length == 0) {
                this.addFileInList();
            }
        },

        addFileInList() {
            this.filesList.push({ type: 1, src: "", name: "", caption: "" });
        },

        deleteItem(index) {
            if (!this.multiple) return;
            this.filesList = this.filesList.filter((obj, ndx) => {
                return ndx !== index;
            });
            this.$emit("input", this.filesList);
        },
        updateItem() {
            this.$forceUpdate();
            this.$emit("input", this.filesList);
        },

        onFileLoad(index) {
            this.filesList[index].src = "";
            if (!this.file) return;
            if (this.file.size > 2 * 1024 * 1024) return;
            this.file.index = index;
            this.readFileToVariable();
        },
        readFileToVariable() {
            var reader = new FileReader();
            reader.onload = (e) => {
                this.file.src = e.target.result;
                this.uploadPhoto({ name: this.file.name, index: this.file.index, src: this.file.src });
            };
            reader.readAsDataURL(this.file);
        },

        uploadPhoto(data) {
            this.filesList[data.index].name = data.name;
            this.filesList[data.index].src = data.src;
            this.$forceUpdate();
            this.$emit("input", this.filesList);
        },
    },
};
</script>

<style>
#field-files .v-btn--fab.v-size--x-small {
    width: 22px;
    height: 22px;
}
#field-files .v-btn--fab.v-size--small {
    width: 32px;
    height: 32px;
}
#field-files .comment {
    width: 100%;
    font-weight: normal;
    font-size: 12px;
    padding: 0 4px;
    line-height: 20px;
    border: 1px solid #777;
    border-radius: 6px;
    margin: -8px -8px 12px 95px;
}
</style>
