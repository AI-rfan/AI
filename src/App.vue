<script setup>
import HelloWorld from './components/HelloWorld.vue'
import {ref, onMounted} from 'vue'
import PocketBase from 'pocketbase'

con
const pb = new PocketBase('http://127.0.0.1:8090')
const hi = "My location is "
const isTrue = ref(false)
const solatAPILink = "https://www.e-solat.gov.my/index.php?r=esolatApi/takwimsolat&period=week&zone=SGR01"
const pokemonAPILink = "https://pokeapi.co/api/v2/pokemon/"

let number = ref(0);
let name = ref("");
let added = ref(false);
let empty = ref(false);
let capitalCities = ref(["Shibuya", "Pinang", "Town", "NewYork City", "Kuala Lumpur", "Hawaii"])
let prayerTime = ref({})//{} - object
let pokemon = ref({})
let pokemonName = ref("")
let todoInput = ref("")
let records = ref([]) 

function increment(){
  number.value++;
  console.log(number);
}

function startInterval(){
  setInterval(increment,100)
}

function addCity(){
  if(name.value != ""){
    capitalCities.value.push(name.value)
    empty.value = false
    added.value = true
    name.value = ""
  } else{
    empty.value = true
  }
}
function getPrayerTime(){
    fetch(solatAPILink).then(result => result.json()).then(data=>{      
      prayerTime.value = data.prayerTime[0]
      console.log(prayerTime.value)
    }
  )
}

function getPokemon(){
    fetch(pokemonAPILink + pokemonName.value).then(result => result.json()).then(data =>{
      pokemon.value = data
      console.log(pokemon.value)
   } 
  )
}

function getTodos() {
  pb.collection('todos').getFullList(200, { sort: '-created' }).then(todos => {
    records.value = todos
  })
}

function createTodo(){
  const newTodo = {
    item: todoInput.value,
    isDone: false
  }
  pb.collection('todos').create(newTodo).then(res => {
    records.value.push(res)
    todoInput.value =""
  })
}

onMounted(()=>{//want to get something when page load
  getPrayerTime()
})

</script>
<template>
  <div class="min-h-screen bg-gradient-to-br from-black to-red-700 p-8">
    <div class="max-w-3xl mx-auto">
      <!-- Counter Section -->
      <div class="bg-black text-white rounded-lg shadow-lg p-6 mb-8">
        <h1 class="text-4xl font-bold text-red-600 mb-4">{{ number }}</h1>
        <button 
          @click="startInterval" 
          class="bg-red-600 hover:bg-red-700 text-white font-medium py-2 px-4 rounded-md transition duration-300 ease-in-out transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-opacity-50">
          +Aura
        </button>
      </div>
      
      <!-- Pocketbase Section -->
      <div class="flex gap-2 mb-4">
          <input type="text" 
            v-model="todoInput" 
            class="flex-1 border border-gray-300 rounded-md px-3 py-2 focus:outline-none focus:ring-2 focus:ring-red-500 focus:border-red-500"
            placeholder="Enter new todo">
          <button 
            @click="createTodo" 
            class="bg-red-600 hover:bg-red-700 text-white font-medium py-2 px-4 rounded-md transition duration-300 ease-in-out focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-opacity-50">
            Add Todo
          </button>
        </div>

        <!-- Display Todo List -->
        <div class="bg-gray-800 text-white rounded-md p-4">
          <h3 class="text-lg font-medium text-gray-300 mb-2">Todos:</h3>
          <ol class="list-decimal list-inside space-y-1">
            <li v-for="todo in records" :key="todo.id" class="text-gray-300 hover:text-red-600 transition duration-200">
              <span>{{ todo.item }}</span>
              <span v-if="todo.isDone" class="text-green-500">(Completed)</span>
              <span v-else class="text-red-500">(Pending)</span>
            </li>
          </ol>
        </div>

      <!-- City Input Section -->
      <div class="bg-black text-white rounded-lg shadow-lg p-6 mb-8">
        <div class="mb-6">
          <label for="cityInput" class="block text-sm font-medium text-gray-300 mb-1">Add a City</label>
          <div class="flex gap-2">
            <input 
              type="text" 
              id="cityInput"
              v-model="name" 
              class="flex-1 border border-gray-300 rounded-md px-3 py-2 focus:outline-none focus:ring-2 focus:ring-red-500 focus:border-red-500"
              placeholder="Enter city name">
            <button 
              @click="addCity" 
              class="bg-red-600 hover:bg-red-700 text-white font-medium py-2 px-4 rounded-md transition duration-300 ease-in-out focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-opacity-50">
              Add City
            </button>
          </div>
        </div>
        
        <div class="mb-4">
          <p v-if="added==true" class="text-red-500 font-medium">City Added Successfully!</p>
          <p v-if="empty==true" class="text-red-500 font-medium">Please Type Anything?</p>
          <h2 class="text-xl font-semibold text-gray-300 mt-2">{{name}}</h2>
        </div>
        
        <h1 class="text-2xl font-bold text-gray-300 mb-3">{{ hi }}</h1>
        
        <div class="bg-gray-800 text-white rounded-md p-4">
          <h3 class="text-lg font-medium text-gray-300 mb-2">Cities:</h3>
          <ol class="list-decimal list-inside space-y-1">
            <li v-for="city in capitalCities" class="text-gray-300 hover:text-red-600 transition duration-200">{{ city }}</li>
          </ol>
        </div>
      </div>
      
      <!-- Toggle Section -->
      <div class="bg-black text-white rounded-lg shadow-lg p-6 mb-8 text-center">
        <button 
          @click="isTrue = !isTrue" 
          class="bg-red-600 hover:bg-red-700 text-white font-medium py-2 px-4 rounded-md transition duration-300 ease-in-out transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-opacity-50 mb-4">
          Toggle Message
        </button>
        <h1 v-if="isTrue" class="text-2xl font-bold text-gray-300 mt-2 animate-pulse">England is my city</h1>
      </div>
      
      <!-- Prayer Time Section -->
      <div class="bg-black text-white rounded-lg shadow-lg p-6 mb-8">
        <div class="flex justify-between items-center mb-4">
          <h2 class="text-xl font-bold text-gray-300">Prayer Times</h2>
          <button 
            @click="getPrayerTime" 
            class="bg-red-600 hover:bg-red-700 text-white font-medium py-2 px-4 rounded-md transition duration-300 ease-in-out focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-opacity-50">
            Get Prayer Time
          </button>
        </div>
        
        <div class="overflow-hidden rounded-lg border border-gray-200">
          <table class="w-full">
            <thead>
              <tr class="bg-gray-800 text-gray-300 text-sm font-medium">
                <td class="py-3 px-2 text-center">Fajr</td>
                <td class="py-3 px-2 text-center">Dhuha</td>
                <td class="py-3 px-2 text-center">Dhuhr</td>
                <td class="py-3 px-2 text-center">Asr</td>
                <td class="py-3 px-2 text-center">Maghrib</td>
                <td class="py-3 px-2 text-center">Isha</td>
              </tr>
            </thead>
            <tbody>
              <tr class="text-gray-300 border-t border-gray-200">
                <td class="py-3 px-2 text-center">{{ prayerTime.fajr }}</td>
                <td class="py-3 px-2 text-center">{{ prayerTime.dhuha }}</td>
                <td class="py-3 px-2 text-center">{{ prayerTime.dhuhr }}</td>
                <td class="py-3 px-2 text-center">{{ prayerTime.asr }}</td>
                <td class="py-3 px-2 text-center">{{ prayerTime.maghrib }}</td>
                <td class="py-3 px-2 text-center">{{ prayerTime.isha }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
      
      <!-- Pokemon Section -->
      <div class="bg-black text-white rounded-lg shadow-lg p-6">
        <h2 class="text-xl font-bold text-gray-300 mb-4">Get a Pokémon's Abilities</h2>
        
        <div class="flex gap-2 mb-6">
          <input 
            type="text" 
            v-model="pokemonName" 
            class="flex-1 border border-gray-300 rounded-md px-3 py-2 focus:outline-none focus:ring-2 focus:ring-red-500 focus:border-red-500"
            placeholder="Enter pokemon name">
          <button 
            @click="getPokemon" 
            class="bg-red-600 hover:bg-red-700 text-white font-medium py-2 px-4 rounded-md transition duration-300 ease-in-out focus:outline-none focus:ring-2 focus:ring-red-500 focus:ring-opacity-50">
            Go
          </button>
        </div>
        
        <div class="overflow-hidden rounded-lg border border-gray-200">
          <table class="w-full">
            <thead>
              <tr class="bg-red-50 text-gray-600 text-sm font-medium">
                <td class="py-3 px-4">Ability Name</td>
              </tr>
            </thead>
            <tbody>
              <tr v-for="ability in pokemon.abilities" :key="ability.ability.name" class="text-gray-700 border-t border-gray-200 hover:bg-red-50 transition duration-150">
                <td class="py-3 px-4 capitalize">{{ ability.ability.name }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</template>

<!--
<template>
  <h1>{{ number }}</h1>
  <button @click="startInterval">+Aura</button>
  <br>
  <br>
    <input type="text" v-model="name">
  <br>
  <h1>{{ hi }}</h1>
  <button @click="addCity">Add City</button>
  <p v-if="added==true" style="color:#39FF14">City Added Succesfully!</p>
  <p v-if="empty==true" style="color:red">Please Type Anything?</p>
  <h2>{{name}}</h2>
  <ol>
    <li v-for="city in capitalCities" style="text-align: center;">{{ city }}</li>
  </ol>
  
  <button @click="isTrue = !isTrue">Click Here</button>
  <br>
  <h1 v-if="isTrue">England is my city</h1>
  <button @click="getPrayerTime">Get Prayer Time</button>
  <br>
  <br>
  <div class="border rounded-sm bg0slate-300 m-2 shadow-md p-2">
  <table class="w-full">
    <thead>
      <tr class="text-center">
        <td>Fajr</td>
        <td>Dhuha</td>
        <td>Dhuhr</td>
        <td>Asr</td>
        <td>Maghrib</td>
        <td>Isha</td>
      </tr>
    </thead>
    <tbody>
      <tr class="text-center">
        <td>{{ prayerTime.fajr }}</td>
        <td>{{ prayerTime.dhuha }}</td>
        <td>{{ prayerTime.dhuhr }}</td>
        <td>{{ prayerTime.asr }}</td>
        <td>{{ prayerTime.maghrib }}</td>
        <td>{{ prayerTime.isha }}</td>
      </tr>
    </tbody>
  </table>
  </div>
  <h2>Get a pokemon's abilities</h2>
  <br>
    <input type="text" v-model="pokemonName">
    <button @click="getPokemon">Go</button>
  <br>
  
  <table>
    <thead>
      <tr>
        <td>Ability Name</td>
      </tr>
    </thead>
    <tbody>
      <tr v-for="ability in pokemon.abilities" :key="ability.ability.name"> //v-for="(item, key, index) in dataSource"> 
        <td>{{ ability.ability.name }}</td>
      </tr>
    </tbody>
  </table>
</template>-->

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
