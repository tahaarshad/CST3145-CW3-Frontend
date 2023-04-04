<template>
  <div id="app">
    <!--Navigation Bar-->
    <nav class="navbar fixed-top navbar-expand-lg navbar-light">
      <div class="container-fluid">
        <div class="navbar-header">
          <a class="navbar-brand" v-on:click="showCheckout"
            ><span class="fa-solid fa-school-flag fs-4"></span>
            {{ sitename }}</a
          >
        </div>
        <a
          class="nav navbar-nav navbar-right"
          v-if="enableCheckout"
          v-on:click="showCheckout"
          ><i class="fa-solid fa-cart-shopping fs-4 position-relative"></i>
          <span class="badge bg-danger border border-light rounded-circle">{{
            cartItemCount
          }}</span>
        </a>
        <a
          class="nav navbar-nav navbar-right disabled"
          v-else
          v-on:click="showCheckout"
          ><i class="fa-solid fa-cart-shopping fs-4 position-relative"></i>
        </a>
      </div>
    </nav>
    <div v-if="showLesson">
      <!--Hero Section-->
      <div class="container text-center py-5 mb-4">
        <div class="p-5 mb-4 lc-block">
          <div class="lc-block mb-4">
            <div editable="rich">
              <h2 class="fw-bold display-2">Extracurricular Activities</h2>
            </div>
          </div>
          <div class="lc-block mb-5">
            <div editable="rich">
              <p class="lead">
                Below are a list of activities taking place after school timings
                for students to expand their skills and knowledge. Confirm spaces now.
              </p>
            </div>
          </div>
          <!--Search-->
          <div class="lc-block mb-2">
            <div class="form-outline">
              <input
                type="search"
                v-model="search"
                v-on:input="searchLessons()"
                class="form-control"
                placeholder="Search Lesson"
              />
              <br />
              <div class="container">
                <!--Sort-->
                <div class="row">
                  <div class="col-md-3">
                    <h4>Sort Lessons:</h4>
                  </div>
                  <div class="col-md-3">
                    <select
                      class="form-select"
                      aria-label="Default select example"
                      @change="sortLessons()"
                      v-model="sortBy"
                    >
                      <option value="Default">--Select Attribute--</option>
                      <option value="Subject">Subject</option>
                      <option value="Location">Location</option>
                      <option value="Price">Price</option>
                      <option value="Availability">Availability</option>
                    </select>
                  </div>
                  <div class="col-md-3">
                    <select
                      class="form-select"
                      aria-label="Default select example"
                      @change="sortLessons()"
                      v-model="sortOrder"
                    >
                      <option value="Ascending">Ascending</option>
                      <option value="Descending">Descending</option>
                    </select>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <!--Lessons-->
      <div class="container">
        <div class="row">
          <LessonList @addToCart="addLesson" :lessons="lessons" />
        </div>
      </div>
    </div>

    <div v-else>
      <!--Hero Section-->
      <div class="container text-center py-5 mb-4">
        <div class="p-5 mb-4 lc-block">
          <div class="lc-block mb-4">
            <div editable="rich">
              <h2 class="fw-bold display-2">Checkout Form</h2>
            </div>
          </div>
          <div class="lc-block mb-5">
            <div editable="rich">
              <p class="lead">Please review you cart and confirm booking.</p>
            </div>
          </div>
        </div>

        <!--Cart-->
        <CartList
          @removeSpace="delSpace"
          @removeItem="delItem"
          @addSpace="incSpace"
          :cart="cart"
        />
        <br />
        <!--Checkout Form-->
        <h3>Checkout</h3>
        <p>
          <strong>Name:</strong>
          <input v-model.trim="name" />
        </p>
        <p>
          <strong>Phone:</strong>
          <input v-model.number="phone" />
        </p>
        <button
          class="submitButton"
          v-if="enableSubmit"
          v-on:click="submitOrder()"
          data-bs-toggle="modal"
          data-bs-target="#confirmSeats"
        >
          Checkout
        </button>
        <button class="submitButton" disabled v-else>Checkout</button>
      </div>
      <!--Confirmation Dialog. Triggered when spaces are booked-->
      <div
        class="modal fade"
        id="confirmSeats"
        tabindex="-1"
        role="dialog"
        aria-labelledby="confirmOrder"
        aria-hidden="true"
      >
        <div class="modal-dialog" role="document">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title">Al Ilm School</h5>
              <button
                type="button"
                class="close"
                data-bs-dismiss="modal"
                aria-label="Close"
              >
                <span aria-hidden="true">&times;</span>
              </button>
            </div>
            <div class="modal-body">
              <p>Seats Confirmed</p>
            </div>
            <div class="modal-footer">
              <button
                type="button"
                class="btn btn-secondary"
                data-bs-dismiss="modal"
              >
                OK
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

import LessonList from "./components/Lessons.vue";
import CartList from "./components/Checkout.vue";

export default {
  name: "App",
  components: { LessonList, CartList },
  data() {
    return {
      sitename: "Al-Ilm School",
      cart: [],
      lessons: [],
      name: "",
      phone: "",
      showLesson: true,
      // Stores user input and selection
      sortBy: "Default",
      sortOrder: "Ascending",
      search: "",
    };
  },
  created: function () {
    //Fetches lesson from MongoDB
    console.log("requesting data from server...");

    fetch("https://alilm-backend.herokuapp.com/collection/lessons").then((response) => {
      response.json().then((data) => {
        this.lessons = data;
        console.log(data);
      });
    });
  },
  methods: {
    // Submits order
    submitOrder() {
      let order = {
        name: this.name,
        phone: this.phone,
        lessons: this.cart,
      };
      fetch("https://alilm-backend.herokuapp.com/collection/orders", {
        method: "POST",
        body: JSON.stringify(order),
        headers: {
          "Content-type": "application/json; charset=UTF-8",
        },
      });

      this.update();
    },
    // Updates records
    update() {
      let updatedLesson = [];
      let updatedSpace = [];
      // let updatedLessons = []
      for (let i = 0; i < this.cart.length; i++) {
        for (let j = 0; j < this.lessons.length; j++) {
          if (this.cart[i].id == this.lessons[j]._id) {
            updatedLesson.push(this.cart[i].id);
            let item = {};
            item.Space = this.lessons[j].Space;
            updatedSpace.push(item);
          }
        }
      }
      console.log(updatedLesson[0]);
      console.log(updatedSpace[0]);
      for (let index = 0; index < updatedLesson.length; index++) {
        let newSpace = JSON.stringify(updatedSpace[index]);
        fetch(
          "https://alilm-backend.herokuapp.com/collection/lessons/" + updatedLesson[index],
          {
            method: "PUT",
            body: newSpace,
            headers: {
              "Content-type": "application/json; charset=UTF-8",
            },
          }
        )
          .then((res) => res.json())
          .then((res) => {
            console.log(res);
          })
          .catch((err) => console.log(err));
      }
      this.cart.splice(0, this.cart.length);
      this.name = "";
      this.phone = "";
    },
    // Checks if the lesson already exists in cart
    getCartItem(lesson) {
      for (let i = 0; i < this.cart.length; i++) {
        if (this.cart[i].id === lesson._id) {
          return this.cart[i];
        }
      }
      return null;
    },
    // Adds lesson to cart
    addLesson(lesson) {
      let cartItem = this.getCartItem(lesson);
      //If item is already in cart, then increase quantity. If not, push lesson to cart
      if (cartItem != null) {
        cartItem.quantity++;
      } else {
        let item = {};
        item.id = lesson._id;
        item.name = lesson.Subject;
        item.price = lesson.Price;
        item.quantity = 1;
        this.cart.push(item);
      }
      lesson.Space--;
    },

    // Removes lesson from cart and adds the quantity back to lesson Space
    delItem(item) {
      for (let i = 0; i < this.lessons.length; i++) {
        if (this.lessons[i]._id === item.id) {
          this.lessons[i].Space += item.quantity;
          let itemIndex = this.cart.indexOf(item);
          this.cart.splice(itemIndex, 1);
        }
      }
    },
    // Removes one space from the item quantity
    delSpace(item) {
      if (item.quantity === 1) {
        this.delItem(item);
      } else {
        for (let i = 0; i < this.lessons.length; i++) {
          if (this.lessons[i]._id === item.id) {
            this.lessons[i].Space++;
            item.quantity--;
          }
        }
      }
      // Adds one space from the item quantity
    },
    incSpace(item) {
      for (let i = 0; i < this.lessons.length; i++) {
        if (this.lessons[i]._id === item.id) {
          this.lessons[i].Space--;
          item.quantity++;
        }
      }
    },
    // Checkout page toggle
    showCheckout: function () {
      this.showLesson = this.showLesson ? false : true;
    },
    // Filters lessons based on user input
    searchLessons() {
      if (this.search.length > 0) {
        fetch(`https://alilm-backend.herokuapp.com/collection/lessons/search/${this.search}`)
          .then((res) => {
            return res.json();
          })
          .then((data) => {
            this.lessons = data;
          })
          .catch((err) => {
            this.lessons = [];
            console.log(`unable to get lessons: ${err}`);
          });
      }
    },
    // Sort Lessons based on user selection
    sortLessons() {
      if (this.sortBy == "Default") {
        this.lessons.sort((a, b) => {
          if (a.id < b.id) {
            return -1;
          }
          if (a.id > b.id) {
            return 1;
          }
          return 0;
        });
      } else if (this.sortBy == "Subject" && this.sortOrder == "Ascending") {
        this.lessons.sort((a, b) => {
          let fa = a.Subject.toLowerCase(),
            fb = b.Subject.toLowerCase();
          if (fa < fb) {
            return -1;
          }
          if (fa > fb) {
            return 1;
          }
          return 0;
        });
      } else if (this.sortBy == "Subject" && this.sortOrder == "Descending") {
        this.lessons.sort((a, b) => {
          let fa = a.Subject.toLowerCase(),
            fb = b.Subject.toLowerCase();
          if (fa < fb) {
            return 1;
          }
          if (fa > fb) {
            return -1;
          }
          return 0;
        });
      } else if (this.sortBy == "Location" && this.sortOrder == "Ascending") {
        this.lessons.sort((a, b) => {
          let fa = a.Location.toLowerCase(),
            fb = b.Location.toLowerCase();
          if (fa < fb) {
            return -1;
          }
          if (fa > fb) {
            return 1;
          }
          return 0;
        });
      } else if (this.sortBy == "Location" && this.sortOrder == "Descending") {
        this.lessons.sort((a, b) => {
          let fa = a.Location.toLowerCase(),
            fb = b.Location.toLowerCase();

          if (fa < fb) {
            return 1;
          }
          if (fa > fb) {
            return -1;
          }
          return 0;
        });
      } else if (this.sortBy == "Price" && this.sortOrder == "Ascending") {
        this.lessons.sort((a, b) => {
          if (a.Price < b.Price) {
            return -1;
          }
          if (a.Price > b.Price) {
            return 1;
          }
          return 0;
        });
      } else if (this.sortBy == "Price" && this.sortOrder == "Descending") {
        this.lessons.sort((a, b) => {
          if (a.Price < b.Price) {
            return 1;
          }
          if (a.Price > b.Price) {
            return -1;
          }
          return 0;
        });
      } else if (
        this.sortBy == "Availability" &&
        this.sortOrder == "Ascending"
      ) {
        this.lessons.sort((a, b) => {
          if (a.Space < b.Space) {
            return -1;
          }
          if (a.Space > b.Space) {
            return 1;
          }
          return 0;
        });
      } else if (
        this.sortBy == "Availability" &&
        this.sortOrder == "Descending"
      ) {
        this.lessons.sort((a, b) => {
          if (a.Space < b.Space) {
            return 1;
          }
          if (a.Space > b.Space) {
            return -1;
          }
          return 0;
        });
      }
    },
  },
  computed: {
    // Returns number of items in cart
    cartItemCount: function () {
      return this.cart.length;
    },
    // Checkout button enables when cart lenght is more than 0
    enableCheckout: function () {
      return this.cart.length > 0;
    },
    // Validates user input using regular expressions
    enableSubmit: function () {
      let isnum = /^\d+$/.test(this.phone);
      let isletter = /^[A-Za-z\s]*$/.test(this.name);
      return isnum == true && isletter == true;
    },
  },
};
</script>
