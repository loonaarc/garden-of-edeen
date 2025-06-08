<template>

  <!-- Main application container with garden background -->
  <main class="min-h-screen w-full overflow-x-auto"
    style="background-image: url('/bg/grass2.png'); background-repeat: repeat; background-size: 5vw auto;">

    <!-- Application header with title -->
    <header class="fixed top-4 right-4 text-4xl font-extrabold text-green-100 drop-shadow-lg z-40">
      <h1>🌸 GARDEN OF EDEEN 🌸</h1>
    </header>

    <!-- Primary navigation/action bar for planting new seeds -->
    <nav class="fixed top-4 left-4 z-40" role="banner">
      <section class="bg-green-200/50 backdrop-blur rounded-2xl p-4 border-2 border-green-300 shadow-lg">
        <!-- Seed planting form -->
        <form @submit.prevent="plantSeed" class="flex gap-3 items-center">

          <!-- Main input for new task/idea -->
          <label class="sr-only" for="seed-title">Enter your idea</label>
          <input id="seed-title" v-model="newSeedTitle" type="text" placeholder="Enter your idea..."
            class="px-4 py-2 rounded-full border-2 border-green-300 focus:border-green-500 outline-none min-w-[200px]"
            aria-describedby="seed-help" />
          <span id="seed-help" class="sr-only">Enter a new task or idea to plant in your garden</span>

          <!-- Category/label selection -->
          <label class="sr-only" for="seed-label">Choose category</label>
          <select id="seed-label" v-model="newSeedLabel"
            class="px-4 py-2 rounded-full border-2 border-green-300 focus:border-green-500 outline-none"
            aria-describedby="label-help">
            <option value="">Choose label...</option>
            <option v-for="label in labels" :key="label" :value="label">{{ label }}</option>
          </select>
          <span id="label-help" class="sr-only">Select a category for your new task</span>

          <!-- Submit button for planting seed -->
          <button type="submit" :disabled="!newSeedTitle.trim()"
            class="bg-green-500 hover:bg-green-600 disabled:bg-gray-400 text-white px-6 py-2 rounded-full font-semibold shadow-lg transition"
            :aria-label="newSeedTitle.trim() ? `Plant seed: ${newSeedTitle}` : 'Enter an idea to plant a seed'">
            Plant Seed 🌱
          </button>
        </form>
      </section>
    </nav>

    <!-- Main content area: Task board grid -->
    <section
      class="grid grid-cols-[auto_repeat(5,minmax(120px,1fr))] gap-0 p-6 pt-24 min-h-[calc(100vh-10rem)] w-full relative"
      style="max-width: 95vw; margin: 0 auto;" role="main" aria-label="Kanban board for task management">

      <!-- Stage column headers - represents task lifecycle stages -->
      <div class="sr-only">Stage Headers</div> <!-- Screen reader context -->
      <div></div> <!-- Empty top-left cell for layout -->

      <!-- Stage header buttons/tags -->
      <header v-for="stage in stages" :key="stage"
        class="bg-pink-500 text-green-50 text-sm font-semibold text-center px-3 py-1 rounded-full mb-4"
        :aria-label="`Stage: ${stage}`">
        {{ stage }}
      </header>

      <!-- Task category rows - each label gets its own row -->
      <template v-for="(label, labelIndex) in labels" :key="label">

        <!-- Category identifier with greenhouse visual -->
        <aside class="flex flex-col items-center pr-4" :aria-label="`Category: ${label}`">
          <!-- Decorative greenhouse image -->
          <figure class="mb-1">
            <img src="/bg/greenhouse3.png?url" alt="Greenhouse representing task category" class="w-32 h-32" />
            <!-- Category label tag -->
            <figcaption class="text-sm font-semibold text-center rounded-full px-2 py-1"
              :style="`background-color: ${getLabelColor(label)}; color: ${getTextColor(getLabelColor(label))};`">
              {{ label }}
            </figcaption>
          </figure>
        </aside>

        <!-- Task cells for each stage within this category -->
        <template v-for="(stage, stageIndex) in stages" :key="label + '-' + stage">
          <section class="relative min-h-[6rem] flex items-center justify-center overflow-visible"
            :aria-label="`${label} tasks in ${stage} stage`">

            <!-- Decorative fence borders for visual garden effect -->
            <!-- Top fence - only shown on first row to avoid duplication -->
            <div v-if="labelIndex === 0" class="absolute -top-2 left-0 w-full h-4 bg-repeat-x z-20 mx-1"
              style="background-image: url('/bg/fence-horizontal.png'); background-size: contain;" aria-hidden="true">
            </div>

            <!-- Bottom fence - always present for consistent borders -->
            <div class="absolute -bottom-2 left-0 w-full h-4 bg-repeat-x z-20 mx-1"
              style="background-image: url('/bg/fence-horizontal.png'); background-size: contain;" aria-hidden="true">
            </div>

            <!-- Left fence - only on first column to avoid duplication -->
            <div v-if="stageIndex === 0" class="absolute top-0 -left-1 h-full w-2 bg-repeat-y z-20 my-1"
              style="background-image: url('/bg/fence-vertical.png'); background-size: contain;" aria-hidden="true">
            </div>

            <!-- Right fence - always present for consistent borders -->
            <div class="absolute top-0 -right-1 h-full w-2 bg-repeat-y z-20 my-1"
              style="background-image: url('/bg/fence-vertical.png'); background-size: contain;" aria-hidden="true">
            </div>

            <!-- Draggable task container - allows moving tasks between stages -->
            <article
              class="relative z-10 flex flex-wrap items-center justify-center gap-1 px-2 py-2 min-h-[4rem] w-full h-full">

              <!-- Vue draggable component for task management -->
              <draggable :list="getFlowersForCell(label, stage)" @change="handleDragChange($event, label, stage)"
                group="flowers" item-key="id" class="flex flex-wrap items-center justify-center gap-1 w-full h-full"
                :aria-label="`Draggable task list for ${label} - ${stage}`">

                <!-- Individual task item template -->
                <template #item="{ element: flower }">
                  <figure class="text-center flex-shrink-0" style="min-width: 60px; max-width: 80px"
                   @mouseover="showTooltip($event, flower)"
    @mouseleave="handleMouseLeave">
                    <!-- Task stage visual representation -->
                    <img :src="getFlowerImage(stage)" :alt="`${stage} stage flower`"
                      class="h-auto w-8 sm:w-10 md:w-11 object-contain mx-auto" />
                    <!-- Task title with category-colored background -->
                    <figcaption
                      class="text-xs sm:text-sm font-medium text-center leading-tight rounded-full px-1 sm:px-2 py-1 break-words"
                      :style="`background-color: ${getLabelColor(label)}; color: ${getTextColor(getLabelColor(label))};`"
                      :title="`Task: ${flower.title} (Category: ${label}, Stage: ${stage})`">
                      {{ flower.title }}
                    </figcaption>

                  </figure>

                </template>

              </draggable>

            </article>

          </section>
        </template>
      </template>

      <!-- Tooltip for hovered flower/task -->
      <div v-if="hoveredFlower" class="absolute bg-white shadow-lg rounded-lg p-4 text-sm z-50"
        :style="{ top: `${tooltipY}px`, left: `${tooltipX}px` }"
  @mouseover="handleMouseEnter"
  @mouseleave="handleMouseLeave">
        <h3 class="font-bold">{{ hoveredFlower.title }}</h3>

        <!-- Editable input field for the title -->
  <input 
    v-model="hoveredFlowerTitle" 
    type="text" 
    class="w-full px-2 py-1 border rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
    placeholder="Enter new title"
  />

        <p><strong>Description:</strong> {{ hoveredFlower.description || 'No description' }}</p>
        <p><strong>Start:</strong> {{ hoveredFlower.startTime || 'N/A' }}</p>
        <p><strong>End:</strong> {{ hoveredFlower.endTime || 'N/A' }}</p>
        <p><strong>Location:</strong> {{ hoveredFlower.locationName || 'N/A' }}</p>


        <!-- Save button -->
  <button 
    @click="saveFlowerTitle" 
    class="mt-2 bg-green-500 hover:bg-green-600 text-white px-4 py-2 rounded-full font-semibold shadow-lg transition"
  >
    Save
  </button>

        <!-- Edit button -->
  <button 
    @click="redirectToEditPage(hoveredFlower.id)" 
    class="mt-2 bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded-full font-semibold shadow-lg transition"
    :aria-label="`Edit ${hoveredFlower.title} in Calendiary`"
  >
    Edit in Calendiary
  </button>
      </div>

      <!-- Add new category section -->
      <aside class="flex flex-col items-center pr-4 mt-4">
        <button @click="showAddLabelModal = true"
          class="bg-purple-500 hover:bg-purple-600 text-white px-4 py-2 rounded-full font-semibold shadow-lg transition flex items-center gap-2"
          aria-label="Add new task category">
          <span class="text-lg" aria-hidden="true">+</span>
          Add Label
        </button>
      </aside>

      <!-- Empty cells to maintain grid layout for add label row -->
      <div v-for="stage in stages" :key="'add-label-' + stage" class="min-h-[6rem] mt-4" aria-hidden="true"></div>
    </section>

    <!-- Layout spacer -->
    <div class="flex justify-center gap-4 mt-8 pb-8" aria-hidden="true">
      <!-- Empty div to maintain layout consistency -->
    </div>

    <!-- Bottom action bar - "Sproutprise" feature for task suggestions -->
    <footer class="fixed bottom-4 left-1/2 transform -translate-x-1/2 z-40">
      <section class="bg-pink-200/50 backdrop-blur rounded-2xl p-4 border-2 border-pink-300 shadow-lg">
        <form @submit.prevent="sproutpriseMe" class="flex gap-4 items-center">
          <span class="text-2xl" aria-hidden="true">🌸</span>

          <!-- Filter selection for task suggestions -->
          <label class="sr-only" for="sprout-filter">Filter suggestions by category</label>
          <select id="sprout-filter" v-model="sproutLabel"
            class="px-4 py-2 rounded-full border-2 border-pink-300 focus:border-pink-500 outline-none"
            aria-describedby="sprout-help">
            <option value="">All labels</option>
            <option v-for="label in labels" :key="label" :value="label">{{ label }}</option>
          </select>
          <span id="sprout-help" class="sr-only">Choose a category to filter task suggestions, or leave blank for all
            categories</span>

          <!-- Suggestion trigger button -->
          <button type="submit"
            class="bg-pink-500 hover:bg-pink-600 text-white px-6 py-2 rounded-full font-semibold shadow-lg transition"
            aria-label="Get a random task suggestion from the Planted stage">
            Sproutprise Me!
          </button>

          <!-- Dynamic suggestion display -->
          <output v-if="suggestion" class="flex items-center gap-2" :aria-live="'polite'">
            <span class="text-pink-800 font-medium">How about</span>
            <div class="backdrop-blur rounded-full px-4 py-2 border-2 border-pink-300 font-medium"
              :style="`background-color: ${getSuggestionLabelColor()}; color: ${getTextColor(getSuggestionLabelColor())};`"
              role="status" :aria-label="`Suggested task: ${suggestion}`">
              {{ suggestion }}
            </div>
            <span class="text-pink-800 font-medium">?</span>
          </output>

          <span class="text-2xl" aria-hidden="true">🌸</span>
        </form>
      </section>
    </footer>

    <!-- Modal dialog for adding new task categories -->
    <dialog v-if="showAddLabelModal" class="fixed inset-0 bg-black/0 flex items-center justify-center z-50"
      :open="showAddLabelModal" aria-labelledby="modal-title" aria-describedby="modal-description">

      <article class="bg-white rounded-2xl p-6 max-w-md mx-4" role="dialog">
        <header>
          <h2 id="modal-title" class="text-xl font-bold mb-4">Add New Label</h2>
          <p id="modal-description" class="sr-only">Create a new category for organizing your tasks</p>
        </header>

        <!-- New category form -->
        <form @submit.prevent="addNewLabel">
          <label class="sr-only" for="new-label-input">New category name</label>
          <input id="new-label-input" v-model="newLabelName" type="text" placeholder="Enter label name..."
            class="w-full px-4 py-2 rounded-full border-2 border-gray-300 focus:border-purple-500 outline-none mb-4"
            aria-describedby="new-label-help" autofocus />
          <span id="new-label-help" class="sr-only">Enter a name for your new task category</span>

          <!-- Modal action buttons -->
          <footer class="flex gap-3 justify-end">
            <button type="button" @click="showAddLabelModal = false"
              class="px-4 py-2 rounded-full border-2 border-gray-300 hover:bg-gray-100 transition"
              aria-label="Cancel adding new category">
              Cancel
            </button>
            <button type="submit" :disabled="!newLabelName.trim()"
              class="bg-purple-500 hover:bg-purple-600 disabled:bg-gray-400 text-white px-4 py-2 rounded-full transition"
              :aria-label="newLabelName.trim() ? `Add category: ${newLabelName}` : 'Enter a name to add new category'">
              Add Label
            </button>
          </footer>
        </form>
      </article>
    </dialog>
  </main>
</template>

<script setup lang="ts">
import draggable from 'vuedraggable'
import { onMounted, reactive, ref } from 'vue'

// Task lifecycle stages - represents the journey from idea to completion
const stages = ['Preserved', 'Planted', 'Growing', 'Harvested', 'Composted']

// Task categories/labels - different areas of life or types of tasks
const labels = reactive<string[]>(['Miscelleanous']);

// Form state management
const newSeedTitle = ref('') // Title for new task/idea
const newSeedLabel = ref('') // Selected category for new task
const sproutLabel = ref('') // Filter for task suggestions
const showAddLabelModal = ref(false) // Modal visibility state
const newLabelName = ref('') // Name for new category
const suggestion = ref('') // Current task suggestion text
const hoveredFlower = ref<Flower | null>(null);
const tooltipX = ref(0);
const tooltipY = ref(0);
const cardVisible = ref(false);
const hoveredFlowerTitle = ref('');
const labelColors = ref<Record<string, string>>({});

let hideTimeout: ReturnType<typeof setTimeout>;

function handleMouseLeave() {
  hideTimeout = setTimeout(() => {
    cardVisible.value = false;
    hoveredFlower.value = null;
  }, 200); // 200ms delay
}

function handleMouseEnter() {
  clearTimeout(hideTimeout); // Cancel the hide timeout
  cardVisible.value = true;
}

function showTooltip(event: MouseEvent, flower: Flower) {
  hoveredFlower.value = flower;
  hoveredFlowerTitle.value = flower.title; // Set the editable title

  // Position the tooltip relative to the flower
  const target = event.currentTarget as HTMLElement;
  const rect = target.getBoundingClientRect();
  tooltipX.value = rect.left + window.scrollX + rect.width / 2; // Center horizontally
  tooltipY.value = rect.top + window.scrollY - 10; // Position slightly above
}



// TypeScript interface for task objects
interface Flower {
  id: string; // Unique identifier
  title: string; // Task title
  label?: string; // Category/label
  stage: string; // Lifecycle stage
  description?: string; // Optional description
  startTime?: string; // Start time (ISO string)
  endTime?: string; // End time (ISO string)
  locationName?: string; // Location name
  fullAddress?: string; // Full address
  diaryEntry?: string; // Diary entry
  moodRating?: number; // Mood rating
}

interface Label {
  id: string;
  name: string;
  color? : string;
}

// Reference to the suggested task object for styling purposes
const suggestionFlower = ref<Flower | null>(null)

// Main data store - all tasks/flowers in the garden
const flowers = reactive<Flower[]>([]);

const jwTToken = ref(localStorage.getItem('jwt')) // JWT token for authentication, if needed

function fetchFlowers() {
  const token = localStorage.getItem('jwt'); // Dynamically retrieve the token
  while (!token) {
    console.error('JWT token not found. Please log in.');
    return;
  }

  fetch('http://localhost:8002/calendar/my-entries', {
    headers: {
      Authorization: `Bearer ${jwTToken.value}`
    }
  })
    .then((response) => {
      if (!response.ok) {
        throw new Error('Failed to fetch flowers. Please check your authentication.');
      }
      return response.json();
    })
    .then((data) => {
      // Filter and transform the data
      const filteredFlowers = data
        .filter((entry: any) => entry.stage) // Only include entries where stage is not null
        .map((entry: any) => ({
          id: entry.id.toString(),
          title: entry.title,
          label: entry.labels?.[0] || 'Miscellaneous', // Use the first label or default to 'Miscellaneous'
          stage: entry.stage,
          description: entry.description,
          startTime: entry.startTime,
          endTime: entry.endTime,
          locationName: entry.locationName,
          fullAddress: entry.fullAddress,
          diaryEntry: entry.diaryEntry,
          moodRating: entry.moodRating,
        }));

      // Populate the flowers array
      flowers.splice(0, flowers.length, ...filteredFlowers);
      console.log('Fetched and populated flowers:', flowers);
    })
    .catch(error => {
      console.error('Error fetching flowers:', error);
    })
}

onMounted(() => {

  const urlParams = new URLSearchParams(window.location.search);
  const token = urlParams.get('token');
  if (token) {
    localStorage.setItem('jwt', token);
    jwTToken.value = token;
  }
  // Fetch labels and flowers
  fetchLabels();
  fetchFlowers();
});

/**
 * Filters tasks for a specific grid cell (category + stage combination)
 * @param label - The category/label to filter by
 * @param stage - The lifecycle stage to filter by
 * @returns Array of tasks matching the criteria
 */
function getFlowersForCell(label: string, stage: string) {
  return flowers.filter(f => f.label === label && f.stage === stage)
}

/**
 * Handles drag and drop events when tasks are moved between cells
 * Updates the task's category and stage based on where it was dropped
 * @param event - The drag event containing moved item information
 * @param targetLabel - The category the item was dropped into
 * @param targetStage - The stage the item was dropped into
 */
function handleDragChange(event: any, targetLabel: string, targetStage: string) {
  if (event.added) {
    // Find the flower that was moved
    const addedFlower = flowers.find(f => f.id === event.added.element.id);
    if (addedFlower) {
      // Update the flower's label and stage locally
      addedFlower.label = targetLabel;
      addedFlower.stage = targetStage;

      // Send a PATCH request to update the flower in the backend
      fetch(`http://localhost:8002/calendar/my-entries/${addedFlower.id}`, {
        method: 'PATCH',
        headers: {
          'Content-Type': 'application/json',
          Authorization: `Bearer ${jwTToken.value}`, // Include JWT token for authentication
        },
        body: JSON.stringify({
          labels: [targetLabel], // Send labels as an array
          stage: targetStage,
        }),
      })
        .then((response) => {
          if (!response.ok) {
            throw new Error('Failed to update flower stage and labels.');
          }
          console.log('Flower stage and labels updated successfully.');
        })
        .catch((error) => {
          console.error('Error updating flower stage and labels:', error);
        });
    }
  }
}

/**
 * Creates a new task and adds it to the garden
 * New tasks always start in the "Planted" stage
 */
function plantSeed() {
  if (!newSeedTitle.value.trim()) return

  // Use selected label or default to 'Miscellaneous'
  const selectedLabel = newSeedLabel.value || 'Miscellaneous'

  // Create new task object
   const newFlower = {
    title: newSeedTitle.value.trim(),
    stage: 'Planted', // All new tasks start as planted
    labels: [selectedLabel],
  };

  // Send POST request to backend
  fetch('http://localhost:8002/calendar/my-entries', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      Authorization: `Bearer ${jwTToken.value}`, // Include JWT token for authentication
    },
    body: JSON.stringify(newFlower),
  })
  .then((response) => {
      if (!response.ok) {
        throw new Error('Failed to plant seed. Please try again.');
      }
      return response.json();
    })
    .then((data) => {
      // Add the newly created flower to the local array
      flowers.push({
        id: data.id.toString(), // Use the ID returned by the backend
        title: data.title,
        stage: data.stage,
        label: data.labels?.[0] || 'Miscellaneous', // Default to 'Miscellaneous' if no label provided
        description: data.description || '',
        startTime: data.startTime || '',
        endTime: data.endTime || '',
        locationName: data.locationName || '',
        fullAddress: data.fullAddress || '',
        diaryEntry: data.diaryEntry || '',
        moodRating: data.moodRating || null,
      });

      console.log('Seed planted successfully:', data);

      // Reset form fields
      newSeedTitle.value = '';
      newSeedLabel.value = '';
    })
    .catch((error) => {
      console.error('Error planting seed:', error);
    });
}

function saveFlowerTitle() {
  if (hoveredFlower.value) {
    // Update the title in the local flowers array
    hoveredFlower.value.title = hoveredFlowerTitle.value;

    // Optionally, send the updated title to the backend
    fetch(`http://localhost:8002/calendar/my-entries/${hoveredFlower.value.id}`, {
      method: 'PATCH',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${jwTToken.value}`,
      },
      body: JSON.stringify({ title: hoveredFlowerTitle.value }),
    })
      .then((response) => {
        if (!response.ok) {
          throw new Error('Failed to update flower title.');
        }
        console.log('Flower title updated successfully.');
      })
      .catch((error) => {
        console.error('Error updating flower title:', error);
      });
  }
}

function redirectToEditPage(flowerId: string) {
  const calendiaryEditUrl = `http://localhost:8080/edit-event/${flowerId}`;
  window.location.href = calendiaryEditUrl;
}

/**
 * Adds a new category/label to the system
 * Prevents duplicate labels and handles form cleanup
 */
function addNewLabel() {
  if (!newLabelName.value.trim()) return;

  const labelName = newLabelName.value.trim();

  // Check for duplicates in the local labels array
  if (labels.includes(labelName)) {
    alert('This label already exists. Please choose a different name.');
    return;
  }

  // Send POST request to backend to create the new label
  fetch('http://localhost:8002/labels', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      Authorization: `Bearer ${jwTToken.value}`, // Include JWT token for authentication
    },
    body: JSON.stringify({ name: labelName }),
  })
    .then((response) => {
      if (!response.ok) {
        throw new Error('Failed to add new label. Please try again.');
      }
      return response.json();
    })
    .then((data) => {
      // Add the new label to the local labels array
      if (!labels.includes(data.name)) {
        labels.push(data.name);
      }

      console.log('Label added successfully:', data);

      // Reset form and close modal
      newLabelName.value = '';
      showAddLabelModal.value = false;
    })
    .catch((error) => {

        console.error('Error adding new label:', error);
        alert('Failed to add new label. Please try again.');
      
    });
}

function fetchLabels() {
  fetch('http://localhost:8002/labels', {
    method: 'GET',
    headers: {
      'Content-Type': 'application/json',
      Authorization: `Bearer ${jwTToken.value}`, // Include JWT token for authentication
    },
  })
    .then((response) => {
      if (!response.ok) {
        throw new Error('Failed to fetch labels. Please check your authentication.');
      }
      return response.json();
    })
    .then((data: Label[]) => {
      // Populate the labels array with data from the backend
      const fetchedLabels = data.map((label) => {
        // Assign a color to the label if not already assigned
        if (!labelColors.value[label.name]) {
          labelColors.value[label.name] = label.color || getRandomColor();
        }
        return label.name;
      });

      // Ensure "Miscellaneous" is included only once
      if (!fetchedLabels.includes('Miscellaneous')) {
        fetchedLabels.unshift('Miscellaneous');
      }

      labels.splice(0, labels.length, ...fetchedLabels);

      console.log('Fetched labels with colors:', labelColors.value);
    })
    .catch((error) => {
      console.error('Error fetching labels:', error);
    });
}

/**
 * "Sproutprise" feature - suggests a random planted task
 * Can be filtered by category or show from all categories
 */
function sproutpriseMe() {
  // Get all planted tasks (tasks ready to be worked on)
  let plantedFlowers = flowers.filter(f => f.stage === 'Planted')

  // Apply category filter if selected
  if (sproutLabel.value) {
    plantedFlowers = plantedFlowers.filter(f => f.label === sproutLabel.value)
  }

  // Handle case where no planted tasks are found
  if (plantedFlowers.length === 0) {
    suggestion.value = 'No planted activities found! Plant some seeds first 🌱'
    suggestionFlower.value = null
    return
  }

  // Select random task and display as suggestion
  const randomIndex = Math.floor(Math.random() * plantedFlowers.length)
  const selectedFlower = plantedFlowers[randomIndex]

  // Store suggestion text and reference for styling
  suggestion.value = selectedFlower.title
  suggestionFlower.value = selectedFlower
}

/**
 * Gets the background color for the current suggestion
 * @returns CSS color string for the suggested task's category
 */
function getSuggestionLabelColor(): string {
  if (!suggestionFlower.value) return 'rgba(255,255,255,0.8)'
  return getLabelColor(suggestionFlower.value.label)
}

function getRandomColor(): string {
  const r = Math.floor(Math.random() * 256);
  const g = Math.floor(Math.random() * 256);
  const b = Math.floor(Math.random() * 256);
  return `rgba(${r}, ${g}, ${b}, 0.7)`; // Random color with transparency
}

/**
 * Maps category labels to distinctive colors
 * @param label - The category name
 * @returns CSS color string with transparency
 */
function getLabelColor(label: string): string {
  if (!labelColors.value[label]) {
    labelColors.value[label] = getRandomColor(); // Assign a random color if not already assigned
  }
  return labelColors.value[label];
}

/**
 * Determines appropriate text color (black/white) based on background
 * Uses brightness calculation to ensure good contrast
 * @param background - CSS background color string
 * @returns '#000' for dark text or '#fff' for light text
 */
function getTextColor(background: string): string {
  const match = background.match(/\d+/g)
  if (!match) return '#000'

  const [r, g, b] = match.map(Number)
  // Calculate perceived brightness using luminance formula
  const brightness = (r * 299 + g * 587 + b * 114) / 1000

  // Return black text for light backgrounds, white for dark
  return brightness > 140 ? '#000' : '#fff'
}

/**
 * Maps lifecycle stages to appropriate flower/plant images
 * @param stage - The current stage of the task
 * @returns Path to the corresponding image file
 */
function getFlowerImage(stage: string): string {
  const map: Record<string, string> = {
    Preserved: '/flowers/preserved.png',   // Ideas kept for later
    Planted: '/flowers/seedbed3.png',      // Ready to start working on
    Growing: '/flowers/growing.png',       // Currently in progress
    Harvested: '/flowers/harvested.png',   // Completed successfully
    Composted: '/flowers/compost.png'      // Abandoned or archived
  }
  return map[stage] || '/flowers/default.png' // Fallback image
}
</script>