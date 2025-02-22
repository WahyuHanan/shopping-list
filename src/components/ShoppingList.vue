<script setup>
import { ref, computed } from "vue";

const newItem = ref("");
const newQuantity = ref(1);
const items = ref([]);
const notification = ref("");
const sort = ref("recent"); // Default sort option
const editingIndex = ref(null);
const editItemName = ref("");
const editItemQuantity = ref(1);

const addItem = () => {
  if (newItem.value.trim() !== "") {
    if (items.value.some((item) => item.text === newItem.value.trim())) {
      notification.value = "This item is already in the list!";
      setTimeout(() => {
        notification.value = "";
      }, 3000); // Clear notification after 3 seconds
      return;
    }
    items.value.push({ text: newItem.value, quantity: newQuantity.value, done: false });
    newItem.value = "";
    newQuantity.value = 1; // Reset quantity after adding
  }
};

const toggleDone = (item) => {
  item.done = !item.done;
};

const deleteItem = (itemToDelete) => {
  items.value = items.value.filter((item) => item.text !== itemToDelete.text);
};

const clearAll = () => {
  items.value = [];
};

const startEdit = (index) => {
  editingIndex.value = index;
  editItemName.value = items.value[index].text;
  editItemQuantity.value = items.value[index].quantity;
};

const saveEdit = (index) => {
  items.value[index].text = editItemName.value;
  items.value[index].quantity = editItemQuantity.value;
  editingIndex.value = null;
};

const cancelEdit = () => {
  editingIndex.value = null;
};

const sortedItems = computed(() => {
  let sorted = [...items.value]; // Create a copy to avoid mutating the original array

  switch (sort.value) {
    case "alphabetical":
      sorted.sort((a, b) => a.text.localeCompare(b.text));
      break;
    case "done":
      sorted.sort((a, b) => (a.done === b.done ? 0 : a.done ? 1 : -1)); // false first, then true
      break;
    default: // "recent"
      // No need to sort, as the items are already in the order they were added
      break;
  }
  return sorted;
});

const handleQuantityEnter = () => {
  addItem();
};
</script>

<template>
  <body class="flex flex-col gap-2">
    <section>
      <div class="container bg-slate-200 rounded-lg p-4 w-3/4 mx-auto mt-8 flex flex-col gap-1">
        <h1 class="text-3xl font-bold underline">Shopping List</h1>
        <div class="flex justify-start items-center -mb-1">
          <h1 class="w-8/10">What to buy:</h1>
          <h1>Qty :</h1>
        </div>

        <div class="buttons flex justify-around items-center">
          <input type="text" v-model="newItem" @keyup.enter="addItem" class="border rounded-2xl text-center w-8/10" />
          <input type="number" id="qty" v-model="newQuantity" @keyup.enter="handleQuantityEnter" class="border rounded-2xl text-center w-2/10" />
        </div>
        <button type="submit" @click="addItem" class="text-white bg-black rounded-4xl hover:font-bold cursor-pointer">Add</button>
        <div v-if="notification" class="text-red-500">{{ notification }}</div>
      </div>
    </section>
    <section>
      <div class="container bg-slate-200 rounded-lg p-4 w-3/4 mx-auto h-[300px] overflow-y-scroll">
        <ul class="list-group">
          <li v-for="(item, index) in sortedItems" :key="item.text" class="list-group-item flex text-xl justify-between items-center">
            <div v-if="editingIndex !== index">
              <input type="checkbox" v-model="item.done" class="mr-2" />
              <span :class="{ 'line-through': item.done }">{{ item.text }} (Qty: {{ item.quantity }})</span>
            </div>
            <div v-else class="bg-slate-300 p-2 rounded-lg flex flex-col justify-around w-3/4">
              <div class="flex flex-row gap-2">
                <label for="rename" class="text-base/tight font-bold">Edit Name:</label>
                <input type="text" v-model="editItemName" class="mr-2 w-3/4 p-2" id="rename" />
              </div>
              <div class="flex flex-row flex-start bg-amber-200 gap-2">
                <label for="rename" class="text-base/tight font-bold">Edit Qty :</label>
                <input type="number" v-model.number="editItemQuantity" class="mr-2 w-20 text-center" />
              </div>
            </div>

            <div v-if="editingIndex !== index">
              <button @click="startEdit(index)" class="text-blue-500 hover:text-blue-700"><i class="bx bx-edit"></i></button>
              <button @click="deleteItem(item)" class="text-red-500 hover:text-red-700"><i class="bx bx-trash"></i></button>
            </div>
            <div v-else class="bg-slate-300 p-1 rounded-lg flex flex-col justify-around items-center">
              <button @click="saveEdit(index)" class="text-green-500 hover:text-green-700"><i class="bx bx-save bx-tada text-xl">Save</i></button>
              <button @click="cancelEdit()" class="text-gray-500 hover:text-gray-700"><i class="bx bx-message-square-x">Cancel</i></button>
            </div>
          </li>
        </ul>
        <button v-if="items.length > 0" class="bg-black text-white font-bold hover:text-red-600 w-20 rounded-2xl flex justify-center place-self-center mt-4" @click="clearAll">Clear All</button>
      </div>
    </section>
    <section class="-mt-4">
      <div class="container bg-slate-200 rounded-lg p-4 w-3/4 mx-auto mt-4">
        <h1>Items to buy: {{ items.length }}</h1>
        <h1>Items bought: {{ items.filter((item) => item.done).length }}</h1>
        <label for="sort">Sort items by: </label>
        <select name="sortItems" id="sort" class="border rounded-2xl" v-model="sort">
          <option value="recent">Recent</option>
          <option value="alphabetical">Alphabetical</option>
          <option value="done">Done</option>
        </select>
      </div>
    </section>
  </body>
</template>

<style scoped>
.line-through {
  text-decoration: line-through;
}
</style>
