<script>
import { computed } from '@vue/reactivity'

export default{
    props: {
        matched: {
            type: Boolean,
            default: false
        },
        position: {
            type: Number,
            required: true
        },
        value: {
            type: String,
            required: true
        },
        visible: {
            type: Boolean,
            default: true
        }
    },
    setup(props, context) {

        const flippedStyles = computed(()=>{
            if(props.visible){
                return 'is-flipped'
            }
        })
        // we want to emit this to the parent
        const selectCard = () => {
            context.emit('select-card', {
                // wich card was selected
                position: props.position,
                faceValue: props.value
            })
        }


        return {
            selectCard,
            flippedStyles
        }
    }
}
</script>

<template>    
    <div class="card" :class="flippedStyles" @click="selectCard">
        <div  class="card-face is-front">

            <img :src="`/images/${value}.png`" :alt="value">

            <img v-if="matched" src="../../public/images/checkmark.svg" class="icon-checkmark" />
            
        </div>
        <div  class="card-face is-back" style="color:black;font-size:2rem;">
        </div>
    </div>
</template>

<style>

.card{
  /* border: 5px solid #ccc; */
  position: relative;
  transition: 0.5s transform ease-in;
  transform-style: preserve-3d;
}

.card.is-flipped {
    /* lets rotate out card along the Y axis to make it flip */
    transform: rotateY(180deg);
}

.card-face{
    width: 100%;
    height: 100%;
    position: absolute;
    border-radius: 10px;
    display:flex;
    align-items: center;
    justify-content: center;
    /* This div element has "backface-visibility: visible",
     and the back face of the div element shows a mirror image of the front face: */
    backface-visibility: hidden;
}

.card-face.is-front{
    /* face down by default */
    color:white;
    background-image: url('../../public/images/card-bg.png');    
    transform: rotateY(180deg);
}

.card-face.is-back{
    /* display:none; */
    background-image: url('../../public/images/card-bg-empty.png');
    color: white;
}

.icon-checkmark {
    position: absolute;
    right: 5px;
    bottom:5px;
}

</style>