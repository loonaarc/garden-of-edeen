<template>
  <div class="min-h-screen w-full overflow-x-auto"
    style="background-image: url('/bg/grass2.png'); background-repeat: repeat; background-size: 5vw auto;">

    <h1 class="fixed top-4 right-4 text-4xl font-extrabold text-green-100 drop-shadow-lg z-40">
      🌸 GARDEN OF EDEEN 🌸
    </h1>

    <!-- Top Action Bar - Plant a Seed -->
    <header class="fixed top-4 left-4 z-40">
      <div class="bg-green-200/50 backdrop-blur rounded-2xl p-4 border-2 border-green-300 shadow-lg">
        <div class="flex gap-3 items-center">
          <span class="text-2xl font-bold text-green-800">🌱</span>
          <input 
            v-model="newSeedTitle" 
            type="text" 
            placeholder="Enter your idea..."
            class="px-4 py-2 rounded-full border-2 border-green-300 focus:border-green-500 outline-none min-w-[200px]"
          />
          <select 
            v-model="newSeedLabel"
            class="px-4 py-2 rounded-full border-2 border-green-300 focus:border-green-500 outline-none"
          >
            <option value="">Choose label...</option>
            <option v-for="label in labels" :key="label" :value="label">{{ label }}</option>
          </select>
          <button 
            @click="plantSeed"
            :disabled="!newSeedTitle.trim()"
            class="bg-green-500 hover:bg-green-600 disabled:bg-gray-400 text-white px-6 py-2 rounded-full font-semibold shadow-lg transition"
          >
            Plant Seed
          </button>
        </div>
      </div>
    </header>

    <div
      class="grid grid-cols-[auto_repeat(5,minmax(120px,1fr))] gap-4 p-6 pt-24 min-h-[calc(100vh-10rem)] w-full"
      style="max-width: 95vw; margin: 0 auto;">

      <!-- Stage headers as tag-like fields -->
      <div></div> <!-- Empty top-left cell -->
      <div v-for="stage in stages" :key="stage"
        class="bg-pink-500 text-green-50 text-sm font-semibold text-center px-3 py-1 rounded-full">
        {{ stage }}
      </div>

      <!-- Rows for each label -->
      <template v-for="label in labels" :key="label">
        <!-- Label cottage with colored label text -->
        <div class="flex flex-col items-center">
          <img src="/bg/greenhouse3.png?url" alt="Greenhouse" class="w-32 h-32 mb-1" />
          <!-- Label tag below cottage -->
          <div class="text-sm font-semibold text-center rounded-full px-2 py-1" :style="`
    background-color: ${getLabelColor(label)};
    color: ${getTextColor(getLabelColor(label))};
  `">
            {{ label }}
          </div>

        </div>

        <!-- Grid cell with fences and flowers -->
        <template v-for="stage in stages" :key="label + '-' + stage">
          <div class="relative min-h-[6rem] flex items-center justify-center overflow-hidden">

            <!-- Fences -->
            <div class="absolute top-0 left-0 w-full h-4 bg-repeat-x"
              style="background-image: url('/bg/fence-horizontal.png'); background-size: contain;"></div>
            <div class="absolute bottom-0 left-0 w-full h-4 bg-repeat-x"
              style="background-image: url('/bg/fence-horizontal.png'); background-size: contain;"></div>
            <div class="absolute top-0 left-0 h-full w-2 bg-repeat-y"
              style="background-image: url('/bg/fence-vertical.png'); background-size: contain;"></div>
            <div class="absolute top-0 right-0 h-full w-2 bg-repeat-y"
              style="background-image: url('/bg/fence-vertical.png'); background-size: contain;"></div>

            <!-- Plot content -->
            <draggable 
              :list="getFlowersForCell(label, stage)"
              @change="handleDragChange($event, label, stage)"
              group="flowers" 
              item-key="id"
              class="relative z-10 flex flex-wrap items-center justify-center gap-1 px-2 py-2 min-h-[4rem]">
              <template #item="{ element: flower }">
                <div class="text-center flex-shrink-0" style="min-width: 60px; max-width: 80px;">
                  <img :src="getFlowerImage(stage)" alt="" class="h-auto w-8 sm:w-10 md:w-11 object-contain mx-auto" />
                  <div class="text-xs sm:text-sm font-medium text-center leading-tight rounded-full px-1 sm:px-2 py-1 break-words"
                    :style="`background-color: ${getLabelColor(label)}; color: ${getTextColor(getLabelColor(label))};`">
                    {{ flower.title }}
                  </div>
                </div>
              </template>
            </draggable>
          </div>
        </template>
      </template>

      <!-- Add Label Button Row -->
      <div class="flex flex-col items-center">
        <button 
          @click="showAddLabelModal = true"
          class="bg-purple-500 hover:bg-purple-600 text-white px-4 py-2 rounded-full font-semibold shadow-lg transition flex items-center gap-2"
        >
          <span class="text-lg">+</span>
          Add Label
        </button>
      </div>
      
      <!-- Empty cells for the add label row -->
      <div v-for="stage in stages" :key="'add-label-' + stage" class="min-h-[6rem]"></div>
    </div>

    <div class="flex justify-center gap-4 mt-8 pb-8">
      <!-- Empty div to maintain layout consistency -->
    </div>

    <!-- Bottom Sproutprise Bar -->
    <div class="fixed bottom-4 left-1/2 transform -translate-x-1/2 z-40">
      <div class="bg-pink-200/50 backdrop-blur rounded-2xl p-4 border-2 border-pink-300 shadow-lg">
        <div class="flex gap-4 items-center">
          <span class="text-2xl">🌸</span>
          <select 
            v-model="sproutLabel"
            class="px-4 py-2 rounded-full border-2 border-pink-300 focus:border-pink-500 outline-none"
          >
            <option value="">All labels</option>
            <option v-for="label in labels" :key="label" :value="label">{{ label }}</option>
          </select>
          <button @click="sproutpriseMe"
            class="bg-pink-500 hover:bg-pink-600 text-white px-6 py-2 rounded-full font-semibold shadow-lg transition">
             Sproutprise Me!
          </button>
          <div v-if="suggestion" class="flex items-center gap-2">
            <span class="text-pink-800 font-medium">How about</span>
            <div class="backdrop-blur rounded-full px-4 py-2 border-2 border-pink-300 font-medium"
                 :style="`background-color: ${getSuggestionLabelColor()}; color: ${getTextColor(getSuggestionLabelColor())};`">
              {{ suggestion }}
            </div>
            <span class="text-pink-800 font-medium">?</span>
          </div>
          <span class="text-2xl">🌸</span>
        </div>
      </div>
    </div>

    <!-- Add Label Modal -->
    <div v-if="showAddLabelModal" class="fixed inset-0 bg-black/50 flex items-center justify-center z-50">
      <div class="bg-white rounded-2xl p-6 max-w-md mx-4">
        <h3 class="text-xl font-bold mb-4">Add New Label</h3>
        <input 
          v-model="newLabelName"
          type="text" 
          placeholder="Enter label name..."
          class="w-full px-4 py-2 rounded-full border-2 border-gray-300 focus:border-purple-500 outline-none mb-4"
          @keyup.enter="addNewLabel"
        />
        <div class="flex gap-3 justify-end">
          <button 
            @click="showAddLabelModal = false"
            class="px-4 py-2 rounded-full border-2 border-gray-300 hover:bg-gray-100 transition"
          >
            Cancel
          </button>
          <button 
            @click="addNewLabel"
            :disabled="!newLabelName.trim()"
            class="bg-purple-500 hover:bg-purple-600 disabled:bg-gray-400 text-white px-4 py-2 rounded-full transition"
          >
            Add Label
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import draggable from 'vuedraggable'
import { reactive, ref } from 'vue'

const stages = ['Preserved', 'Planted', 'Growing', 'Blossomed', 'Composted']
const labels = reactive(['Miscellaneous', 'Friends', 'Chores', 'Health', 'Learning', 'Fun'])

// Form states
const newSeedTitle = ref('')
const newSeedLabel = ref('')
const sproutLabel = ref('')
const showAddLabelModal = ref(false)
const newLabelName = ref('')
const suggestion = ref('')
interface Flower {
  id: string
  title: string
  label: string
  stage: string
}
const suggestionFlower = ref<Flower | null>(null)

// Blumen als reaktive Liste
const flowers = reactive([
  { id: '1', title: 'Write a poem', label: 'Chores', stage: 'Planted' },
  { id: '2', title: 'Go to gym', label: 'Health', stage: 'Growing' },
  { id: '3', title: 'Game night', label: 'Friends', stage: 'Preserved' },
  { id: '4', title: 'Watch documentary', label: 'Learning', stage: 'Composted' },
  { id: '5', title: 'Go to Northpole', label: 'Fun', stage: 'Blossomed' },
  { id: '6', title: 'Walk cat', label: 'Fun', stage: 'Planted' },
  { id: '7', title: 'Drink bubble tea', label: 'Friends', stage: 'Preserved' },
  { id: '8', title: 'Think about ideas', label: 'Miscellaneous', stage: 'Growing' },
  { id: '9', title: 'Buy milk', label: 'Chores', stage: 'Growing' },
  { id: '10', title: 'Physiotherapy appointment', label: 'Chores', stage: 'Preserved' }
])

// Returns a plain array for the current cell
function getFlowersForCell(label: string, stage: string) {
  return flowers.filter(f => f.label === label && f.stage === stage)
}

// Handle drag and drop changes
function handleDragChange(event: any, targetLabel: string, targetStage: string) {
  if (event.added) {
    // Item was added to this cell
    const addedFlower = flowers.find(f => f.id === event.added.element.id)
    if (addedFlower) {
      addedFlower.label = targetLabel
      addedFlower.stage = targetStage
    }
  }
}

// Plant a new seed
function plantSeed() {
  if (!newSeedTitle.value.trim()) return
  
  const selectedLabel = newSeedLabel.value || 'Miscellaneous'
  
  flowers.push({
    id: Math.random().toString(36).substr(2, 9),
    title: newSeedTitle.value.trim(),
    stage: 'Planted',
    label: selectedLabel
  })
  
  // Reset form
  newSeedTitle.value = ''
  newSeedLabel.value = ''
}

// Add new label
function addNewLabel() {
  if (!newLabelName.value.trim()) return
  
  const labelName = newLabelName.value.trim()
  if (!labels.includes(labelName)) {
    labels.push(labelName)
  }
  
  // Reset form and close modal
  newLabelName.value = ''
  showAddLabelModal.value = false
}

function sproutpriseMe() {
  // Get all planted flowers based on label selection
  let plantedFlowers = flowers.filter(f => f.stage === 'Planted')
  
  if (sproutLabel.value) {
    plantedFlowers = plantedFlowers.filter(f => f.label === sproutLabel.value)
  }
  
  if (plantedFlowers.length === 0) {
    suggestion.value = 'No planted activities found! Plant some seeds first 🌱'
    suggestionFlower.value = null
    return
  }
  
  // Pick a random planted flower and show as suggestion
  const randomIndex = Math.floor(Math.random() * plantedFlowers.length)
  const selectedFlower = plantedFlowers[randomIndex]
  
  // Show the suggestion and store the flower for color reference
  suggestion.value = selectedFlower.title
  suggestionFlower.value = selectedFlower
}

function getSuggestionLabelColor(): string {
  if (!suggestionFlower.value) return 'rgba(255,255,255,0.8)'
  return getLabelColor(suggestionFlower.value.label)
}

function getLabelColor(label: string): string {
  const colorMap: Record<string, string> = {
    Work: 'rgba(255, 99, 99, 0.7)',
    Chores: 'rgba(216, 127, 255, 0.7)',
    Health: 'rgba(110, 255, 110, 0.7)',
    Learning: 'rgba(135, 206, 250, 0.7)',
    Fun: 'rgba(255, 239, 112, 0.7)',
    Friends: 'rgba(255, 192, 203, 0.7)',
    Miscellaneous: 'rgba(169, 169, 169, 0.7)'
  }
  return colorMap[label] || 'rgba(255,255,255,0.5)'
}

function getTextColor(background: string): string {
  const match = background.match(/\d+/g)
  if (!match) return '#000'
  const [r, g, b] = match.map(Number)
  const brightness = (r * 299 + g * 587 + b * 114) / 1000
  return brightness > 140 ? '#000' : '#fff'
}

function getFlowerImage(stage: string): string {
  const map: Record<string, string> = {
    Preserved: '/flowers/preserved.png',
    Planted: '/flowers/seedbed3.png',
    Growing: '/flowers/growing.png',
    Blossomed: '/flowers/harvested.png',
    Composted: '/flowers/compost.png'
  }
  return map[stage] || '/flowers/default.png'
}
</script>