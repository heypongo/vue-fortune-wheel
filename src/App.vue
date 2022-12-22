<template>
  <div class="wrap" @click="handleClick">
    <Wheel
      ref="wheel"
      :gift="gift"
      :data="data"
      @done="done"
      :img-src="logo"
      :is-clicked="isClicked"
    />
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";
import Wheel from "./components/Wheel.vue";

export default defineComponent({
  name: "App",
  components: {
    Wheel
  },
  setup() {
    const isClicked = ref(false);

    return {
      isClicked,
    };
  },
  data() {
    return {
      gift: 5,
      data: [
        {
          id: 1,
          value: "Gift 1",
          bgColor: "#7d7db3",
          color: "#ffffff",
        },
        {
          id: 2,
          value: "Gift 2",
          bgColor: "#ffffff",
          color: "#000000",
        },
        {
          id: 3,
          value: "Gift 3",
          bgColor: "#c92729",
          color: "#ffffff",
        },
        {
          id: 4,
          value: "Gift 4",
          bgColor: "#7d7db3",
          color: "#ffffff",
        },
        {
          id: 5,
          value: "Gift 5",
          bgColor: "#ffffff",
          color: "#000000",
        },
        {
          id: 6,
          value: "Gift 6",
          bgColor: "#c92729",
          color: "#ffffff",
        },
      ],
      logo: "https://upload.wikimedia.org/wikipedia/commons/thumb/9/95/Vue.js_Logo_2.svg/2367px-Vue.js_Logo_2.svg.png",
    };
  },
  methods: {
    done(r) {
      console.log(r);
    },
    handleClick() {
      this.isClicked = true;
    },
  },
});
</script>

<style>
.wrap {
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
