<template>
    <div id="field-photos" class="mb-4">
        <v-card v-if="isAvatar" :style="'width:' + width + '; height:' + height + ';'">
            <v-img v-if="filesList.length > 0 && !disabled" :src="filesList[0].src" contain style="width: 100%; height: 100%; cursor: pointer" @click="$refs.file.click()" />
            <v-img v-if="filesList.length > 0 && disabled" :src="filesList[0].src" contain style="width: 100%; height: 100%;" />
            <input type="file" ref="file" v-show="false" @change="avatarSelected" accept="image/*" />
        </v-card>

        <v-card v-else>
            <v-sheet class="py-1 px-4 body-1 white--text d-flex justify-space-between" color="primary">
                <div class="title">{{ label }}</div>
                <v-btn v-if="multiple" fab small color="green" dark @click.stop="addPhotoInList()"><v-icon>add</v-icon></v-btn>
            </v-sheet>

            <v-card-title class="py-0" v-for="(item, i) in filesList" :key="i">
                <template v-if="item.src && item.src.length>9">
                    <!-- Отображаем загруженный файл  -->
                    <img :src="item.src" style="max-height: 50px; max-width: 50px; margin: 0 10px 0 0" />
                    <a class="filelink" :href="item.src" target="_blank">{{ item.name }}</a>
                    <v-spacer />
                    <input class="comment my-2" placeholder="Описание файла" v-model="item.caption" @input="updateItem()" :disabled="disabled" />
                    <v-btn class="mx-2 my-2" small fab color="green" dark @click.stop="cropItem(i)" v-if="!disabled"><v-icon>edit</v-icon></v-btn>
                    <v-divider class="mx-1" vertical />
                    <v-btn class="ml-2 my-2" small fab color="red" dark @click.stop="deleteItem(i)" v-if="!disabled"><v-icon>close</v-icon></v-btn>
                </template>
                <template v-else>
                    <v-switch dense color="gray" v-model="item.type" :true-value="2" :false-value="1" :label="item.type == 1 ? 'файл' : ' URL '"></v-switch>
                    <v-btn v-if="multiple" class="ml-1" x-small fab color="red" dark @click.stop="deleteItem(i)"><v-icon>close</v-icon></v-btn>
                    <template v-if="item.type == 1">
                        <v-file-input
                            class="ml-1"
                            chips
                            label="Файл фотографии"
                            v-model="rawFiles[i]"
                            accept="image/*"
                            show-size
                            :rules="[(v) => !v || (v && !v.size) || (v && v.size < 2 * 1024 * 1024) || 'Размер файла должен быть не более 2 MB']"
                            prepend-icon=""
                            @change="onFileItemChange(i)"
                            hide-details
                            outlined
                            dense
                            :error="hasError"
                            :disabled="disabled"
                        ></v-file-input>
                    </template>
                    <template v-else>
                        <v-text-field class="ml-1" v-model="item.src" label="URL фотографии" hide-details outlined dense @input="updateItem()" :disabled="disabled"></v-text-field>
                    </template>

                    <img :src="item.src" style="max-height: 50px; max-width: 50px; margin: -4px -6px -4px 2px; float: right" />
                    <input v-if="item.src.length > 9" class="comment" placeholder="Комментарий" v-model="item.caption" @input="updateItem()" :disabled="disabled" />
                </template>
            </v-card-title>

            <v-divider v-if="multiple" />
            <v-card-actions v-if="multiple" class="py-0">
                <v-spacer />
                <v-btn fab small color="green" class="ma-1" dark @click.stop="addPhotoInList" :disabled="disabled"><v-icon>add</v-icon></v-btn>
            </v-card-actions>
        </v-card>

        <v-dialog v-model="dialogCropper.active" scrollable style="width:calc(100vw - 50px); height:calc(100vh - 20px)">
            <v-card v-if="dialogCropper.active">
                <v-card-text class="pa-0 d-flex" style="overflow: hidden;">
                    <div style="width: 50px; background:#116; padding:8px">
                        <v-btn @click="cropperResize(2,2)" class="my-1" icon><v-icon color="white">mdi-fullscreen</v-icon></v-btn>
                        <v-btn @click="cropperResize(0.5,0.5)" class="my-1" icon><v-icon color="white">mdi-fullscreen-exit</v-icon></v-btn>

                        <v-btn @click="$refs.cropper.zoom(2)" class="my-1" icon><v-icon color="white">mdi-magnify-plus-outline</v-icon></v-btn>
                        <v-btn @click="$refs.cropper.zoom(0.5)" class="my-1" icon><v-icon color="white">mdi-magnify-minus-outline</v-icon></v-btn>
                        <v-btn @click="$refs.cropper.flip(true, false)" class="my-1" icon><v-icon color="white">mdi-flip-horizontal</v-icon></v-btn>
                        <v-btn @click="$refs.cropper.flip(false, true)" class="my-1" icon><v-icon color="white">mdi-flip-vertical</v-icon></v-btn>
                        <v-btn @click="$refs.cropper.rotate(-45)" class="my-1" icon><v-icon color="white">mdi-rotate-left</v-icon></v-btn>
                        <v-btn @click="$refs.cropper.rotate(45)" class="my-1" icon><v-icon color="white">mdi-rotate-right</v-icon></v-btn>
                        <v-btn @click="showDialogResize" class="my-1" icon><v-icon color="white">mdi-arrow-top-right-bottom-left</v-icon></v-btn>
                    </div>
                    <cropper
                        class="cropper"
                        ref="cropper"
                        :src="filesList[dialogCropper.fileIndex].src"
                        :stencil-props="{
                        }"
                        :canvas="{
				width: limitations.width,
				height: limitations.height,
			}"
                    />
                </v-card-text>

                <v-card-actions style="background:#116">
                    <v-btn @click="dialogCropper.active = false" color="red">Закрыть</v-btn>
                    <v-spacer />
                    <v-btn @click="cropperSave" color="green">Применить</v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>

        <v-dialog v-model="dialogResize.active" width="320">
            <v-card>
                <v-card-title>Размер изображения <v-spacer /> <v-btn icon @click="dialogResize.active=false"><v-icon>close</v-icon></v-btn> </v-card-title>
                <v-card-text>
                    <v-text-field label="Ширина (Width)" v-model="limitations.width" dense hide-details :outlined="true" />
                    <v-text-field label="Высота (Height)" v-model="limitations.height" dense hide-details :outlined="true" class="pt-4" />
                </v-card-text>
                <v-card-actions>
                    <v-spacer />
                    <v-btn color="primary" @click="dialogResize.active=false"><v-icon left>save</v-icon> Применить </v-btn>
                    <v-spacer />
                </v-card-actions>
            </v-card>
        </v-dialog>

    </div>
</template>

<script>
import { Cropper } from 'vue-advanced-cropper';
import 'vue-advanced-cropper/dist/style.css';

export default {
    props: {
        isAvatar: { type: Boolean, default: false },
        width: { type: String, default: "100%" },
        height: { type: String, default: "100px" },

        label: { type: String, default: null },
        value: { required: true },
        multiple: { type: Boolean, default: false },
        rules: { type: Array, default: null },
        disabled: { default: false },
    },
    data() {
        return {
            filesList: [],
            rawFiles: [],
            hasError: false,
            limitations:{width:800, height:800},
            dialogCropper:{active:false, fileIndex:0},
            dialogResize:{active:false},
        };
    },
    watch: {
        value() {
            this.refresh();
        },
    },
    mounted() {
        this.refresh();
        if (this.rules && this.$parent.$parent.register) this.$parent.$parent.register(this); //for check validate();
    },
    beforeDestroy() {
        if (this.rules && this.$parent.$parent.register) this.$parent.$parent.unregister(this);
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



        onFileItemChange(index) {
            this.filesList[index].src = "";
            if (!this.rawFiles[index]) return;
            if (this.rawFiles[index].size > 3 * 1024 * 1024) return;
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
        async fileLoaded(data) {
            this.filesList[data.index].name = data.name;
            this.filesList[data.index].src = await this.resizeDataURL(data.src, 800, 800);
            this.updateItem();
        },
        async resizeDataURL(datas, maxW, maxH) {
	    let isJpeg = datas.substr(0, 15) == "data:image/jpeg";
            return new Promise((resolve, reject) => {
                var img = document.createElement("img");
                img.onload = function() {
                    let canvas = document.createElement("canvas");
                    let ctx = canvas.getContext("2d");
                    let cw = img.width;
                    let ch = img.height;
                    let scale = Math.min(maxW / cw, maxH / ch);
                    if (scale < 1) {
                        canvas.width = cw * scale;
                        canvas.height = ch * scale;
                        ctx.drawImage(this, 0, 0, canvas.width, canvas.height);
                        const dataURI = isJpeg ? canvas.toDataURL("image/jpeg", 0.75) : canvas.toDataURL();
                        resolve(dataURI);
                    } else {
                        resolve(datas);
                    }
                };
                img.src = datas;
            });
        },

        showDialogResize(){
            this.dialogResize.active = true;
        },

        cropItem(index) {
            this.dialogCropper.active = true;
            this.dialogCropper.fileIndex = index;
            setTimeout(()=>{
                const { canvas, image } = this.$refs.cropper.getResult();
                this.limitations.width = image.width;
                this.limitations.height = image.height;
                //console.log(this.limitations);
            },900);
        },

        cropperSave() {
            this.dialogCropper.active = false;
            const { coordinates, canvas, } = this.$refs.cropper.getResult();
            //console.log(coordinates, canvas)
            this.filesList[this.dialogCropper.fileIndex].src = canvas.toDataURL();            
        },
        cropperResize(width = 1, height = 1) {
			let startCoordinates;
			this.$refs.cropper.setCoordinates([
				({ coordinates, imageSize }) => {
					startCoordinates = coordinates;
					return {
						width: coordinates.width * width,
						height: coordinates.height * height,
					};
				},
				({ coordinates, imageSize }) => ({
					left: startCoordinates.left + (startCoordinates.width - coordinates.width) / 2,
					top: startCoordinates.top + (startCoordinates.height - coordinates.height) / 2,
				}),
			]);
		},

        validate() {
            this.hasError = false;
            if (this.disabled) return true;
            if (this.rules) {
                for (let index = 0; index < this.rules.length; index++) {
                    const rule = this.rules[index];
                    const valid = typeof rule === 'function' ? rule(this.filesList) : rule;
          
                    if (valid === false || typeof valid === 'string') {
                        this.hasError = true;
                        return false;
                    }
                }
            }
            this.hasError = false;
            return true;
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

.cropper {
    width:calc(100vw - 50px); 
    height:calc(100vh - 20px);	
    background: #000;
}
</style>

