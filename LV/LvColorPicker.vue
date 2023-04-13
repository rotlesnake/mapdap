<template>
    <div id="lv-color-picker mb-4">
        <div class="pt-1">Выбор цвета</div>
        <input ref="color_picker" class="color-picker" :value="color" data-huebee @focus="onFocus" @blur="onBlur" />
    </div>
</template>

<script>
import Huebee from "../libs/huebee/huebee.js";
import "../libs/huebee/huebee.css";

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
            this.updateColor();
        },
    },

    mounted(){
        this.updateColor();
        setTimeout(()=>{
            if (!this.$refs.color_picker) return;
            this.updateColor();
            var hueb = new Huebee( this.$refs.color_picker, {
              setBGColor: true,
              saturations: 2,
            });

            hueb.on( 'change', ( color, hue, sat, lum )=>{
                this.save();
            })

            hueb.close();
        }, 300);
    },

    methods: {
        updateColor() {
            this.color = this.value;
        },
        save() {
            this.color = this.$refs.color_picker.value;
            this.$emit('input', this.color)
        },

        onFocus(evt){
            evt.target.style.height='180px';
        },
        onBlur(evt){
            evt.target.style.height='auto';
        },
    },
}
</script>

<style>
.color-picker { 
    border: 1px solid #999;
    border-radius: 6px;
 }
</style>
