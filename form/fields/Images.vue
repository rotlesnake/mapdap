<template>
    <div id="field-photos" class="mb-8">
        <v-card v-if="isAvatar" :style="'width:' + width + '; height:' + height + ';'">
            <v-img v-if="filesList.length > 0" :src="filesList[0].src" contain style="width:100%; height:100%; cursor:pointer" @click="$refs.avatarFile.click()" />
            <input type="file" ref="rawFiles[0]" v-show="false" @change="avatarSelected" accept="image/*" />
        </v-card>

        <v-card v-else>
            <v-sheet class="py-2 px-4 body-1 white--text d-flex justify-space-between" color="primary">
                <div class="title">{{ label }}</div>
                <v-btn v-if="multiple" fab small color="green" dark @click.stop="addItem()"><v-icon>add</v-icon></v-btn>
            </v-sheet>

            <v-card-title class="py-0" v-for="(item, i) in filesList" :key="i">
                <v-switch dense color="gray" v-model="item.type" :true-value="2" :false-value="1" :label="item.type == 1 ? 'файл' : ' URL '"></v-switch>

                <template v-if="item.type == 1">
                    <v-file-input
                        class="ml-1"
                        chips
                        label="Файл фотографии"
                        v-model="rawFiles[i]"
                        accept="image/*"
                        show-size
                        :rules="[
                            (v) =>
                                !v ||
                                (v && !v.size) ||
                                (v && v.size < 2 * 1024 * 1024) ||
                                'Размер файла должен быть не более 2 MB',
                        ]"
                        prepend-icon=""
                        @change="onFileItemChange(i)"
                        hide-details
                        outlined
                        dense
                    ></v-file-input>
                    <v-btn v-if="multiple" class="mx-1" x-small fab color="red" dark @click.stop="deleteItem(i)"><v-icon>close</v-icon></v-btn>
                </template>
                <template v-else>
                    <v-text-field
                        class="ml-1"
                        v-model="item.src"
                        label="URL фотографии"
                        hide-details
                        outlined
                        dense
                        @input="updateItem()"
                    ></v-text-field>
                </template>

                <img :src="item.src" style="max-height:50px; max-width:50px; margin:-4px -6px -4px 2px; float:right;" />
                <input
                    v-if="item.src.length > 9"
                    class="comment"
                    placeholder="Комментарий"
                    v-model="item.caption"
                    @input="updateItem()"
                />

            </v-card-title>
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
            if (!this.multiple && this.filesList.length == 0) {
                this.addItem();
            }
        },

        addItem() {
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

        onFileItemChange(index) {
            this.filesList[index].src = "";
            if (!this.rawFiles[index]) return;
            if (this.rawFiles[index].size > 2 * 1024 * 1024) return;
            this.rawFiles[index].index = index;

            this.readFileToVariable( this.rawFiles[index] );
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


        avatarSelected(event) {
            if (!event.target.files) return;
            this.rawFiles[0] = event.target.files[0];
            var reader = new FileReader();
            reader.onload = (e) => {
                this.rawFiles[0].src = e.target.result;
                this.filesList[0].name = this.rawFiles[0].name;
                this.filesList[0].src = this.rawFiles[0].src;
                this.updateItem();
            };
            reader.readAsDataURL(this.rawFiles[0]);
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
    clear: both;
    width: 100%;
    font-weight: normal;
    font-size: 10px;
    padding: 0 4px;
    line-height: 18px;
    outline: 0;
    border: 1px solid #777;
    border-radius: 4px;
    margin: -8px 90px 12px 100px;
}
</style>
