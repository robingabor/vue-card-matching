<template>
<h1 class="sr-only" >Peek-a-vue</h1>
<img src="../public/images/peek-a-vue-title.png" alt="Peek-a-Vue" class="title">
<section class="description">
  <p>
    Welcome to Peek a Vue
  </p>
  <p>
    A card matching game powered by Vue.js 3
  </p>
</section>

  <transition-group tag="section" class="game-board" name="shuffle-card">
    <CardComp v-for="card in cardList" 
      :key="`${card.value}-${card.variant}`"
      :matched="card.matched"
      :value="card.value"
      :visible="card.visible"
      @select-card="flipCard"
      :position="card.position"
    />
  </transition-group > 

  <h2 class="status" >Status: {{ status }}</h2>

  <button v-if="newPlayer" class="button"  @click="startGame"> <img src="/images/play.svg" alt="start">  Start Game</button>

  <button v-else class="button"  @click="restartGame"> <img src="/images/restart.svg" alt="restart">  Restart Game</button>

</template>

<script>
import _ from 'lodash'
import { launchConfetti} from './utilities/confetti'
import { computed, ref, watch } from 'vue'
import CardComp from './components/CardComp.vue'

export default {
    name: "App",
    components: {
    CardComp
  },
  setup(){
    const cardList = ref([])
    // track the selected cards
    const userSelection = ref([])
    const newPlayer = ref(true)

    const startGame = () => {
      newPlayer.value = false

      restartGame()

    }

    const status = computed(() => {
      if(remainingPairs.value === 0 ){
        return 'Player Wins'
      }else{
        return `Remaining Pairs: ${remainingPairs.value}`
      }
    })
    // track the remaining pairs
    const remainingPairs = computed(() => {
      const remainingCards = cardList.value.filter(
        card => card.matched === false
      ).length
      return remainingCards / 2
    })


    const restartGame = () => {
      // reassing the cards new position
      cardList.value = _.shuffle(cardList.value)

      cardList.value = cardList.value.map((card,index) => {
        return {
          // spread ... unpacks the elements of an iterable object
          // rest ... packs the elements into an array
          ...card,          
          matched: false,
          position: index,
          visible: false,
        }
      })
    }
    
    const cardItems = [
      'bat',
      'candy',
      'cauldron',
      'cupcake',
      'ghost',
      'moon',
      'pumpkin',
      'witch-hat'
      ]

    cardItems.forEach(item => {
      cardList.value.push({
        value: item,
        variant: 1,
        visible: false,
        position: null,
        matched: false
      })
      cardList.value.push({
        value: item,
        variant: 2,
        visible: true,
        position: null,
        matched: false
      })
    })

    // assigning posiiton
    cardList.value = cardList.value.map((card,index) => {
      return {
        // JS Object Spread
        ...card,
        position: index
      }
    })
   

    const flipCard = payload => {
      // we going to use our position from the child comp here with the help of 'payload'
      cardList.value[payload.position].visible = true;

      if(userSelection.value[0]){
        if(userSelection.value[0].position === payload.position ){
          return
        }else{
          // otherwise assign the second variable
          userSelection.value[1] = payload
        }
           
      }else{
        userSelection.value[0]= payload
      }         
    }
    // watch remaining pairs and launch confetti
    watch(remainingPairs, (currentValue) => {
      if (currentValue === 0 ){

        launchConfetti()

      }
    })

    watch(userSelection, (currentValue) => {
      // console.log(currentValue)
      if (currentValue.length === 2){
        // console.log("Thats it")
        const cardOne = currentValue[0]
        const cardTwo = currentValue[1]

        // check if the cards are matching or not
        if(cardOne.faceValue === cardTwo.faceValue){

          cardList.value[cardOne.position].matched = true;
          cardList.value[cardTwo.position].matched = true;
        }else{
          setTimeout(()=> {
            cardList.value[cardOne.position].visible = false;
            cardList.value[cardTwo.position].visible = false;
          },2000)
          
        }       
        // clear the userSelection array
        userSelection.value.length = 0
      }
    },{ deep: true})

    return {      
      cardList,
      newPlayer,
      startGame,
      flipCard,
      userSelection,
      status,
      restartGame
    }
  }
}
</script>

<style>
html, body {
  margin: 0;
  padding: 0;
}

h1 {
  margin-top: 0;
}

#app {
  background-image: url('../public/images/page-bg.png');
  background-color: #00070c;
  height: 100vh;
  padding-top: 60px;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  padding-top: 60px;
  padding-bottom: 200px;
  color: #fff;
}

.description {
  font-family: 'Titillium Web', sans-serif;
  font-size: 1.2rem;
}

.description p{
  margin:0;
}

.description p:last-child{
  margin-bottom: 30px;
}

.status {
  font-family: 'Titillium Web', sans-serif;
}

.button {
  font-family: 'Titillium Web', sans-serif;
  font-weight: bold;
  font-size: 1.2rem;
  background-color: orange;
  color:white;
  padding: 0.75rem 1rem;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto;
  border: 0;  
  border-radius: 10px;
}

.button img{
  padding-right: 5px;
}

.button:hover{
  cursor: pointer;
}

.game-board{
  display: grid;
  justify-content: center;
  /* margin:auto;
  max-width: 640px; */
  grid-template-columns: repeat(4,120px);
  grid-column-gap:24px;
  grid-template-rows:repeat(4,120px);
  grid-row-gap:24px;  
}

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow:hidden;
  clip: rect(0,0,0,0);
  border: 0;
}

.title{
  padding-bottom: 30px;
}


.shuffle-card-move{
  transition: transform 0.8s ease-in ;

}
</style>
