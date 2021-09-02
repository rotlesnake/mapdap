<template>
<div id="lv-color-picker">
    <v-menu v-model="colorPicker" bottom allow-overflow origin="center center" transition="scale-transition" :close-on-content-click="false" z-index="250">
        <template v-slot:activator="{ on }">
            <v-card class="my-3">
                <v-card-title class="py-1 ">
                    <span class="body-1 pt-1">{{label}}</span>
                    <v-spacer />
                    <v-btn :color="value" v-on="on" width="100" elevation="4" :dark="dark">{{value || 'Стандарт'}}</v-btn>
                </v-card-title>
            </v-card>
        </template>
        <v-card>
            <v-card-text class="pa-1 d-flex justify-space-around align-center">
                <v-color-picker :value="value" @update:color="update" mode="hexa"></v-color-picker>
                <v-btn color="primary" x-large @click.stop="save()">Выбрать</v-btn>
            </v-card-text>
        </v-card>
    </v-menu>
</div>
</template>

<script>
export default {
    props: {
        label: String,
        value: String,
        dark: {type:Boolean, default:false}
    },
    data() {
        return {
			color:'',
            colorPicker: false,
        }
    },
    watch:{
        value(){
            this.color = this.value;
        },
    },

    methods: {
        update(color) {
			this.color = color.hex;
        },
        save() {
			this.$emit('input', this.color)
			this.colorPicker = false;
        },
    },
}
</script>

<style>

</style>
