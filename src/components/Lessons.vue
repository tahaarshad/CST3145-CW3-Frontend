<!--Commponent for displaying lessons-->
<template>
  <div v-for="lesson in lessons" :key="lesson._id" class="col-md-3">
    <div class="card">
      <img class="card-img-top" v-bind:src="lesson.Image" />
      <div class="card-body">
        <h5 class="card-title" v-text="lesson.Subject"></h5>
      </div>
      <ul class="list-group list-group-flush">
        <li class="list-group-item">
          <span v-text="lesson.Location"></span>
        </li>
        <li class="list-group-item"><span v-text="lesson.Price"></span> AED</li>
        <li class="list-group-item">
          <span v-text="lesson.Space"></span> Seats Available
        </li>
      </ul>
      <div class="card-body">
        <button
          class="addButton"
          v-on:click="addToCart(lesson)"
          v-if="lesson.Space > 0"
        >
          Add to Cart
        </button>
        <button class="addButton" v-else disabled>Add to Cart</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "LessonList",
  props: ["lessons"],
  //Added to prevent warning
  emits: ["addToCart"],
  data() {
    return {};
  },
  methods: {
    addToCart(lesson) {
      this.$emit("addToCart", lesson);
    },
  },
};
</script>
