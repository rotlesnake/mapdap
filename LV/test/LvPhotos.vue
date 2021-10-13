<template>
<div id="lv-photos">

    <v-card>
        <v-card-title class="py-0 mb-2">
            <v-spacer />{{title}}<v-spacer />
        </v-card-title>

        <v-card-title class="py-0 mt-n3" v-for="(item,i) in filesList" :key="i">
            <v-switch dense color="gray" v-model="item.type" :label="(item.type==0?'файл':'url')"></v-switch>
            <v-btn class="ml-1" x-small fab color="red" dark @click.stop="deleteUrl(i)"><v-icon>close</v-icon></v-btn>

            <template v-if="item.type===false">
                <v-file-input class="ml-1" style="max-width:250px;" chips label="Файл фотографии" v-model="file" accept="image/*" show-size 
                        :rules="[(v)=>(!v) || (v && !v.size) || (v && v.size < 2*1024*1024) || 'Размер файла должен быть не более 2 MB']" 
                        prepend-icon=""
                        @change="onFileLoad(i)" hide-details outlined dense></v-file-input>
            </template>
            <template v-else>
                <v-text-field class="ml-1" v-model="item.url" label="URL фотографии" hide-details outlined dense @input="updateUrl()"></v-text-field>
            </template>  

            <img :src="item.url" style="max-height:42px; min-width:42px; margin:-4px -6px -4px 2px; float:right;">
            <input v-if="item.url.length>9" class="comment" placeholder="Комментарий" v-model="item.caption" @input="updateUrl()" />
        </v-card-title>

        <v-divider />
        <v-card-actions class="py-0">
            <v-spacer />
            <v-btn fab small color="green" class="ma-1" dark @click.stop="addPhotoInList"><v-icon>add</v-icon></v-btn>
        </v-card-actions>
    </v-card>

</div>
</template>

<script>
export default {
    props: {
        title: String,
        label: String,
        value: String,
    },
    data() {
        return {
            filesList:[],
			file:[],
        }
    },
    watch:{
        value(){
            this.refresh();
        },
    },
    mounted(){
        this.refresh();
    },

    methods: {
        refresh(){
            if (this.value && this.value.length>2) this.filesList = JSON.parse(this.value);
        },

        addPhotoInList(){
            this.filesList.push({type:1, url:'', caption:''});
        },

        deleteUrl(index) {
            this.filesList = this.filesList.filter((obj, ndx)=>{ return ndx!==index; });
	        this.$emit('input', JSON.stringify(this.filesList) )
        },
        updateUrl() {
	        this.$emit('input', JSON.stringify(this.filesList) )
        },


        onFileLoad(index){
            this.filesList[index].url = '';
            if (!this.file) return;
            if (this.file.size>2*1024*1024) return;
            this.file.index = index;
            this.readFileToVariable();
        },
        readFileToVariable() {
            var reader = new FileReader();
            reader.onload = (e) => {
                this.file.src = e.target.result;
                this.uploadPhoto({name:this.file.name, index:this.file.index, src:this.file.src});
            };
            reader.readAsDataURL(this.file);
        },

        uploadPhoto(data) {
            this.$store.commit("SHOW_LOADER", true);
            this.$api("constructor", "upload", "index", data).then(response=>{
                this.$store.commit("SHOW_LOADER", false);
                if (response.error==0) {
                    this.filesList[response.index].url = response.url;
                    this.updateUrl();
                }
            }).catch(e=>{
                this.$store.commit("SHOW_LOADER", false);
                this.$swal.fire({title: 'Ошибка загрузки', icon:'error'});
            });
        },


    },
}
</script>

<style>
#lv-photos .v-btn--fab.v-size--x-small { width:22px; height:22px; }
#lv-photos .v-btn--fab.v-size--small { width:32px; height:32px; }
#lv-photos .comment {width:100%; font-weight:normal; font-size:12px; padding:0 4px; line-height: 20px; border:1px solid #777; border-radius:6px; margin:-8px -8px 12px 95px;}
</style>
