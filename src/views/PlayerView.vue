<template>
  <div class="player">
    <section class="section container is-max-desktop">
      <div class="box">
        <h1 class="title is-4 mb-2">Add Player</h1>
        <form @submit.prevent="addPlayer">
          <div class="field is-horizontal">
            <div class="field-body">
              <div class="field">
                <div class="control has-icons-left">
                  <input
                    :class="{
                      'input is-normal': !v$.name.$error,
                      'input is-danger': v$.name.$error,
                    }"
                    type="text"
                    v-model="name"
                    placeholder="Name"
                  />
                  <span class="icon is-left">
                    <i class="fa fa-user"></i>
                  </span>
                </div>
                <p
                  class="help is-danger"
                  v-for="error in v$.name.$errors"
                  :key="error.$uid"
                >
                  {{ error.$message }}
                </p>
              </div>
              <div class="field">
                <div class="control has-icons-left">
                  <input
                    :class="{
                      'input is-normal': !v$.number.$error,
                      'input is-danger': v$.number.$error,
                    }"
                    type="text"
                    v-model="number"
                    placeholder="Number"
                  />
                  <span class="icon is-left">
                    <i class="fa fa-tag"></i>
                  </span>
                </div>
                <p
                  class="help is-danger"
                  v-for="error in v$.number.$errors"
                  :key="error.$uid"
                >
                  {{ error.$message }}
                </p>
              </div>
            </div>
          </div>
          <div class="field">
            <div class="control">
              <div class="select">
                <select v:model="position">
                  <option>Player</option>
                  <option>Goalie</option>
                </select>
              </div>
            </div>
          </div>
          <div class="field is-grouped is-grouped-right">
            <div class="control">
              <button class="button is-success" type="submit">Submit</button>
            </div>
          </div>
        </form>
      </div>
      <div v-if="isLoading">Players loading...</div>
      <div v-else>
        <List :title="'Players'" :object="players" :childs="childs" />
      </div>
    </section>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import { usePlayerStore } from "@/store/playerStore";
import { Player } from "@/model/player";
import { storeToRefs } from "pinia";
import { useVuelidate } from "@vuelidate/core";
import { required, helpers, maxLength } from "@vuelidate/validators";

const store = usePlayerStore();
const { players } = storeToRefs(store);
const isLoading = ref(true);

const name = ref("");
const number = ref("");
const position = ref("Player");

const numericLimit = (value: any) => {
  if (!value) {
    return true;
  }
  const numericValue = parseFloat(value);
  return numericValue >= 1 && numericValue <= 99;
};

const customAlpha = (value: any) => {
  if (!value) {
    return true;
  }
 
  return /^[a-zA-Z\s]+$/.test(value);
};

const rules = {
  name: { required, customAlpha, maxLength: maxLength(50) },
  number: {
    required,
    numeric: helpers.withMessage(
      "Value must be between 1 and 99",
      numericLimit
    ),
  },
};

const v$ = useVuelidate(rules, { name, number });

const deletePlayer = (player: Player) => {
  store.deletePlayer(player._id, player._rev);
};

const addChild = (icon: string, func: Function | null) => {
  return { icon, func };
};

const childs = [
  addChild("fa fa-pencil", null),
  addChild("fa fa-trash", deletePlayer),
];

const addPlayer = async () => {
  const result = await v$.value.$validate();
  if (result) {
    const newPlayer: Player = {
      name: name.value,
      number: parseFloat(number.value),
      position: position.value,
      _id: "",
      _rev: "",
    };

    store.addPlayer(newPlayer);

    name.value = "";
    number.value = "";
    position.value = "Player";
    v$.value.$reset();
  }
};

onMounted(async () => {
  try {
    await store.fetchPlayers();
    isLoading.value = false;
  } catch (error) {
    console.error(error);
  }
});
</script>
