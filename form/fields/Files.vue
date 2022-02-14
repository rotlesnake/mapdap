<template>
    <div id="field-files" class="mb-8">
        <v-card>
            <v-sheet class="py-1 px-4 body-1 white--text d-flex justify-space-between" color="primary">
                <div class="title">{{ label }}</div>
                <v-btn v-if="multiple" fab small color="green" dark @click.stop="addItem()"><v-icon>add</v-icon></v-btn>
            </v-sheet>

            <v-card-title class="py-0" v-for="(item, i) in filesList" :key="i">
                <template v-if="item.src">
                    <!-- Отображаем загруженный файл  -->
                    <a class="filelink" :href="item.src" target="_blank">{{ item.name }}</a>
                    <v-spacer />
                    <input class="comment" placeholder="Описание файла" v-model="item.caption" @input="updateItem()" />
                    <v-btn class="ml-2 my-2" small fab color="red" dark @click.stop="deleteItem(i)"><v-icon>close</v-icon></v-btn>
                </template>
                <template v-else>
                    <!-- Форма загрузки файла -->
                    <v-switch dense color="gray" v-model="item.type" :true-value="2" :false-value="1" :label="item.type == 1 ? 'файл' : ' URL '"></v-switch>
                    <template v-if="item.type == 1">
                        <v-file-input
                            class="ml-1"
                            chips
                            label="Файл"
                            v-model="rawFiles[i]"
                            accept="image/*"
                            show-size
                            :rules="[(v) => !v || (v && !v.size) || (v && v.size < 35 * 1024 * 1024) || 'Размер файла должен быть не более 35 MB']"
                            prepend-icon=""
                            @change="onFileItemChange(i)"
                            hide-details
                            outlined
                            dense
                        ></v-file-input>
                        <v-btn v-if="multiple" class="mx-1" x-small fab color="red" dark @click.stop="deleteItem(i)"><v-icon>close</v-icon></v-btn>
                    </template>
                    <template v-else>
                        <v-text-field class="ml-1" v-model="item.src" label="URL фотографии" hide-details outlined dense @input="updateItem()"></v-text-field>
                    </template>

                    <input v-if="item.src.length > 9" class="comment" placeholder="Комментарий" v-model="item.caption" @input="updateItem()" />
                </template>
            </v-card-title>

            <v-divider v-if="multiple" />
            <v-card-actions v-if="multiple" class="py-0">
                <v-spacer />
                <v-btn fab small color="green" class="ma-1 mr-2" dark @click.stop="addItem()"><v-icon>add</v-icon></v-btn>
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
            rawFiles: [],
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
                } catch (e) {
                    this.filesList = [];
                }
            } else {
                this.filesList = this.value || [];
                if (!this.value) this.addItem();
            }
            if (this.filesList.length == 0) {
                this.addItem();
            }
        },

        addItem() {
            this.filesList.push({ type: 1, src: "", name: "", caption: "" });
        },
        deleteItem(index) {
            this.filesList = this.filesList.filter((obj, ndx) => {
                return ndx !== index;
            });
            if (this.filesList.length == 0) this.addItem();
            this.$emit("input", this.filesList);
        },
        updateItem() {
            this.$forceUpdate();
            this.$emit("input", this.filesList);
        },

        onFileItemChange(index) {
            this.filesList[index].src = "";
            if (!this.rawFiles[index]) return;
            if (this.rawFiles[index].size > 2 * 1024 * 1024) return;
            this.rawFiles[index].index = index;

            this.readFileToVariable(this.rawFiles[index]);
        },
        readFileToVariable(fileObject) {
            var reader = new FileReader();
            reader.onload = (e) => {
                fileObject.src = e.target.result;
                this.fileLoaded({ name: fileObject.name, index: fileObject.index, src: fileObject.src });
            };
            reader.readAsDataURL(fileObject);
        },
        fileLoaded(data) {
            this.filesList[data.index].name = data.name;
            this.filesList[data.index].src = data.src;
            this.updateItem();
        },
    },
};
</script>

<style scoped>
#field-files .v-btn--fab.v-size--x-small {
    width: 22px;
    height: 22px;
}
#field-files .v-btn--fab.v-size--small {
    width: 32px;
    height: 32px;
}
#field-files .comment {
    width: 50%;
    font-weight: normal;
    font-size: 12px;
    padding-left: 10px;
    border: 1px solid #777;
    border-radius: 4px;
}

#field-files .filelink {
    font-size: 16px;
}
</style>
