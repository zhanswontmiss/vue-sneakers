<script setup>
import { ref, computed, inject } from 'vue';
import axios from 'axios';

import DrawerHead from './DrawerHead.vue';
import CartItemList from './CartItemList.vue';
import InfoBlock from './InfoBlock.vue';

const props = defineProps({
  totalPrice: Number,
  vatPrice: Number,
})

const { cart, closeDrawer } = inject('cart');

const isCreating = ref(false);
const orderId = ref(null);

const createOrder = async() => {
  try {
    isCreating.value = true;

    const { data } = await axios.post('https://ee2d5e5cf100a5be.mokky.dev/orders', {
      items: cart.value,
      totalPrice: props.totalPrice,
    })

    cart.value = [];

    orderId.value = data.id;
  } catch (err) {
    console.log(err);
  } finally {
    isCreating.value = false;
  }
}

const cartIsEmpty = computed(() => cart.value.length === 0);
const buttonDisabled = computed(() => isCreating.value || cartIsEmpty.value)
</script>

<template>
  <div class="fixed top-0 left-0 h-full w-full bg-black z-10 opacity-70"></div>
  <div class="bg-white w-96 h-full fixed right-0 top-0 z-20 p-8">
    <DrawerHead />

    <div v-if="!totalPrice || orderId" class="flex h-full items-center">
      <InfoBlock
        v-if="!totalPrice && !orderId"
        title="Cart is empty"
        description="Add at least one item to your cart"
        image-url="/package-icon.png"
      />

      <InfoBlock
        v-if="orderId"
        title="Order is created!"
        :description="`Your order #${orderId} will be delivered soon.`"
        image-url="/order-success-icon.png"
      />
    </div>

    <div v-else>
      <CartItemList />

      <div class="flex flex-col gap-4 mt-7">
        <div class="flex gap-2">
          <span>Overall:</span>
          <div class="flex-1 border-b border-dashed"></div>
          <b>{{ totalPrice }} ₸</b>
        </div>

        <div class="flex gap-2">
          <span>Tax 5%</span>
          <div class="flex-1 border-b border-dashed"></div>
          <b>{{ vatPrice }} ₸</b>
        </div>

        <button
          :disabled="buttonDisabled"
          @click="createOrder"
          class="mt-4 bg-lime-500 w-full rounded-xl py-3 text-white disabled:bg-slate-400 hover:bg-lime-600 transition active:bg-lime-700 cursor-pointer"
        >
          Order
        </button>
      </div>
    </div>

  </div>
</template>
