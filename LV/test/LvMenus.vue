<template>
<div id="lv-menus">

    <v-card>
        <v-card-title class="py-0 mb-2">
            <v-spacer />{{title}}<v-spacer />
        </v-card-title>

        <draggable v-model="menuList" group="*" @start="drag=true" @end="drag=false" handle=".handle" @change="updateItem()">
            <div v-for="(item,i) in menuList" :key="i"  class="d-flex mx-2">
                <v-icon class="handle">swap_vert</v-icon>
                <v-text-field class="py-1 mx-1" v-model="item.text" label="Название пункта меню" :append-icon="item.newline ? 'reorder' : 'view_module'" @click:append="setNewline(item)"
                            hide-details outlined dense @input="updateItem()">
                </v-text-field>
                <v-icon color="red" @click.stop="deleteItem(i)">close</v-icon>
            </div>
        </draggable>


        <v-divider />
        <v-card-actions class="py-0">
            <v-spacer />
            <v-btn fab small color="green" class="ma-1" dark @click.stop="addItemInList"><v-icon>add</v-icon></v-btn>
        </v-card-actions>
    </v-card>

</div>
</template>

<script>
import draggable from 'vuedraggable'

export default {
    components: {
        draggable,
    },    
    props: {
        title: String,
        label: String,
        value: String,
    },
    data() {
        return {
            menuList:[],
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
            if (this.value && this.value.length>2) {
                this.menuList = JSON.parse(this.value);
                this.menuList.forEach(e=>{ if (!e.newline) e.newline=false; });
            }
        },

        addItemInList(){
            this.menuList.push({text:'', newline:true});
        },

        deleteItem(index) {
            this.menuList = this.menuList.filter((obj, ndx)=>{ return ndx!==index; });
	        this.$emit('input', JSON.stringify(this.menuList) )
        },
        updateItem() {
	        this.$emit('input', JSON.stringify(this.menuList) )
        },
        setNewline(item) {
            item.newline = !item.newline;
	        this.$emit('input', JSON.stringify(this.menuList) )
        },

    },
}
</script>

<style>
#lv-menus .handle {cursor:move}
</style>
