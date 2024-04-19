<script setup>
import { ref, watch } from 'vue';

const isLoading = ref(true);
const items = ref([]);
const search = ref("");
let results = ref([]);
let theid = ref();
let isItemChosen = ref(false);

// get api with included search attribute
const getItems = () => {
  fetch(`https://test-46cda-default-rtdb.europe-west1.firebasedatabase.app/items.json`, {
    method: 'GET'
  })
    .then((rawData) => {
      return rawData.json();
    })
    .then((data) => {
      items.value = data.kashmir
      results = items.value.filter(obj => {
        return obj.name.toLowerCase().includes(search.value.toLowerCase()) || obj.color.toLowerCase() === search.value.toLowerCase();
      })
      items.value = results;
    })
    .finally(() => {
      isLoading.value = false;
    });

}

// call getItems on search change
watch(search, () => {
  isItemChosen.value = false;
  getItems();
});

// gets id from clicked item and stores it
const chosenItem = (id) => {
  getItems();
  isItemChosen.value = true;
  return theid = id;
}
</script>

<template>
  <input class="search-field" type="text" v-model="search"></input>
  <p v-if="isLoading"></p>
  <div v-else>
    <ul class="result-list">
      <!-- iterates data from array and spits them out in a li -->
      <li v-for="item in items" :key="item.itemId">
        <!-- checks if item has been clicked and has correct id -> then spit out item description section -->
        <div v-if="isItemChosen && theid==item.itemId">
          <div class="grid-desc-container">
            <div class="img-desc">
              <img v-bind:src="item.itemImage" />
            </div>
            <div class="flex-desc-container">
              <div class="flex-desc-item desc-name">
                {{ item.name }}
              </div>
              <div class="flex-desc-item item-no">
                Vare-nr: {{ item.itemId }}
              </div>
              <div v-if="item.isStock" class="flex-desc-item item-stock">
                Lager: <span>✓</span>
              </div>
              <div v-else class="flex-desc-item item-stock">
                Lager: <span>⨉</span>
              </div>
              <div v-if="!item.isDiscount" class="flex-desc-item">
                <span class="bold-head-price">Pris</span> <br><span class="bold-price">{{ item.price }}</span> kr
              </div>
              <div v-else class="flex-desc-item">
                <span class="bold-head-price">Pris</span> <br><del class="margin-right-price"> {{ item.price }} kr </del>
                <span class="bold-price">{{ item.discountPrice }}</span> kr
              </div>
              <div class="flex-desc-item bold-head-price">
                Butiksplads: {{ item.positionInStore }}
              </div>
              <div class="flex-desc-item">
                Farve: {{ item.color }}
              </div>
              <div class="flex-desc-item">
                Beskrivelse <p>{{ item.itemDescription }}</p>
              </div>
            </div>
          </div>
        </div>
        <!-- create html elements for each loop iteration -->
        <div v-if="!isItemChosen" v-on:click="chosenItem(item.itemId)" class="grid-container">
          <div class="img-div">
            <img v-bind:src="item.itemImage" />
          </div>
          <div class="flex-container">
            <div class="flex-item result-name"> {{ item.name }}</div>
            <div class="flex-item" v-if="item.stockCount<10">
              lager: <span class="red-text"> {{ item.stockCount }} </span>
            </div>
            <div class="flex-item" v-else-if="item.stockCount>25">
              lager: <span class="green-text"> {{ item.stockCount }} </span>
            </div>
            <div class="flex-item" v-else>
              lager: <span class="orange-text"> {{ item.stockCount }} </span>
            </div>
            <div class="flex-item"> plads: {{ item.positionInStore }} </div>
            <div v-if="!item.isDiscount" class="flex-item item-price"> {{ item.price }} kr</div>
            <div v-else class="flex-item item-price"> <del class="small-text">{{ item.price }} kr</del> {{ item.discountPrice }} kr</div>
          </div>
        </div>
      </li>
    </ul>

  </div>
</template>
