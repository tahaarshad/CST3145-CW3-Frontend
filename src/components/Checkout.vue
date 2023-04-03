<template>
  <h3>Cart</h3>
  <table>
    <thead class="table-head">
      <tr>
        <th>Lesson</th>
        <th>Space</th>
        <th>Total Price (AED)</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="item in cart">
        <td>{{ item.name }}</td>
        <td>
          <button class="addMinus" v-on:click="removeSpace(item)">
            <i class="fa-solid fa-minus"></i>
          </button>
          {{ item.quantity }}
          <button
            :disabled="item.quantity == 5"
            class="addMinus"
            v-on:click="addSpace(item)"
          >
            <i class="fa-solid fa-plus"></i>
          </button>
        </td>
        <td>{{ item.price * item.quantity }}</td>
        <td>
          <a v-on:click="removeItem(item)"><i class="fa-solid fa-trash"></i></a>
        </td>
      </tr>
    </tbody>
    <tfoot>
      <tr class="summary">
        <td class="sum" colspan="3">Sum:</td>
        <td>{{ subTotal }}</td>
      </tr>
    </tfoot>
  </table>
</template>

<script>
export default {
  name: "CartList",
  props: ["cart"],
  emits: ["removeSpace", "removeItem", "addSpace"],
  data() {
    return {};
  },
  methods: {
    removeSpace(item) {
      this.$emit("removeSpace", item);
    },
    removeItem(item) {
      this.$emit("removeItem", item);
    },
    addSpace(item) {
      this.$emit("addSpace", item);
    },
  },
  computed: {
    // Calculates total value of cart
    subTotal: function () {
      let subTotal = 0;
      for (let i = 0; i < this.cart.length; i++) {
        subTotal += this.cart[i].quantity * this.cart[i].price;
      }
      return subTotal;
    },
  },
};
</script>
