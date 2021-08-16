<template>
<div id="lv-condition">

    <v-card>
        <v-card-title class="py-0 mb-2">
            <v-spacer />{{title}}<v-spacer />
        </v-card-title>

        <draggable v-model="conditionList" group="*" @start="drag=true" @end="drag=false" handle=".handle" @change="updateItem()">
            <div v-for="(item,i) in conditionList" :key="i"  class="d-flex mx-2">
                <v-icon class="handle">swap_vert</v-icon>
                <v-text-field class="py-1 mx-1" v-model="item.text" label="Название" hide-details outlined dense @input="updateItem()"></v-text-field>
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
            conditionList:[],
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
                this.conditionList = JSON.parse(this.value);
                this.conditionList.forEach(e=>{ if (!e.newline) e.newline=false; });
            }
        },

        addItemInList(){
            this.conditionList.push({text:''});
        },

        deleteItem(index) {
            this.conditionList = this.conditionList.filter((obj, ndx)=>{ return ndx!==index; });
	        this.$emit('input', JSON.stringify(this.conditionList) )
        },
        updateItem() {
	        this.$emit('input', JSON.stringify(this.conditionList) )
        },

    },
}
</script>

<style>
#lv-condition .handle {cursor:move}
</style>
