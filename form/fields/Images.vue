<template>
    <div id="field-photos" class="mb-4">
        <v-card v-if="isAvatar" :style="'width:' + width + '; height:' + height + ';'">
            <v-img v-if="filesList.length > 0" :src="filesList[0].src" contain style="width: 100%; height: 100%; cursor: pointer" @click="$refs.file.click()" />
            <input type="file" ref="file" v-show="false" @change="avatarSelected" accept="image/*" />
        </v-card>

        <v-card v-else>
            <v-sheet class="my-1 py-2 px-4 body-1 white--text" color="primary">
                {{ label }}
            </v-sheet>

            <v-card-title class="py-0 mt-n1" v-for="(item, i) in filesList" :key="i">
                <template v-if="item.src">
                    <!-- Отображаем загруженный файл  -->
                    <img :src="item.src" style="max-height: 50px; max-width: 50px; margin: 0 10px 0 0" />
                    <a class="filelink" :href="item.src" target="_blank">{{ item.name }}</a>
                    <v-spacer />
                    <input class="comment" placeholder="Описание файла" v-model="item.caption" @input="updateItem()" />
                    <v-btn class="ml-2 my-2" small fab color="red" dark @click.stop="deleteItem(i)"><v-icon>close</v-icon></v-btn>
                </template>
                <template v-else>
                    <v-switch dense color="gray" v-model="item.type" :true-value="2" :false-value="1" :label="item.type == 1 ? 'файл' : ' URL '"></v-switch>
                    <v-btn v-if="multiple" class="ml-1" x-small fab color="red" dark @click.stop="deleteItem(i)"><v-icon>close</v-icon></v-btn>
                    <template v-if="item.type == 1">
                        <v-file-input
                            class="ml-1"
                            chips
                            label="Файл фотографии"
                            v-model="file"
                            accept="image/*"
                            show-size
                            :rules="[(v) => !v || (v && !v.size) || (v && v.size < 2 * 1024 * 1024) || 'Размер файла должен быть не более 2 MB']"
                            prepend-icon=""
                            @change="onFileLoad(i)"
                            hide-details
                            outlined
                            dense
                        ></v-file-input>
                    </template>
                    <template v-else>
                        <v-text-field class="ml-1" v-model="item.src" label="URL фотографии" hide-details outlined dense @input="updateItem()"></v-text-field>
                    </template>

                    <img :src="item.src" style="max-height: 50px; max-width: 50px; margin: -4px -6px -4px 2px; float: right" />
                    <input v-if="item.src.length > 9" class="comment" placeholder="Комментарий" v-model="item.caption" @input="updateItem()" />
                </template>
            </v-card-title>

            <v-divider v-if="multiple" />
            <v-card-actions v-if="multiple" class="py-0">
                <v-spacer />
                <v-btn fab small color="green" class="ma-1" dark @click.stop="addPhotoInList"><v-icon>add</v-icon></v-btn>
            </v-card-actions>
        </v-card>
    </div>
</template>

<script>
export default {
    props: {
        isAvatar: { type: Boolean, default: false },
        width: { type: String, default: "100%" },
        height: { type: String, default: "100px" },

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
                } catch (e) {
                    this.filesList = [];
                }
            } else {
                this.filesList = this.value || [];
                if (!this.value) this.addPhotoInList();
            }
            if (this.filesList.length == 0) {
                this.addPhotoInList();
            }
        },

        addPhotoInList() {
            this.filesList.push({ type: 1, src: "", name: "", caption: "" });
        },

        deleteItem(index) {
            this.filesList = this.filesList.filter((obj, ndx) => {
                return ndx !== index;
            });
            if (this.filesList.length == 0) this.addPhotoInList();
            this.$emit("input", this.filesList);
        },
        updateItem() {
            this.$forceUpdate();
            this.$emit("input", this.filesList);
        },

        avatarSelected(event) {
            if (!event.target.files) return;
            this.file = event.target.files[0];
            this.onFileLoad(0);
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
#field-photos .v-btn--fab.v-size--x-small {
    width: 22px;
    height: 22px;
}
#field-photos .v-btn--fab.v-size--small {
    width: 32px;
    height: 32px;
}
#field-photos .comment {
    width: 50%;
    font-weight: normal;
    font-size: 12px;
    padding-left: 10px;
    border: 1px solid #777;
    border-radius: 4px;
}
</style>
