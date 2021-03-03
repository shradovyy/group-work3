<template>
    <div class="cart hideCart">
        <div class="cart-content">

            <button @click="displayCart(false)" class="cart-btn close-cart"><i class="fas fa-times"></i></button>
            <div class="title">Total: £{{ cartTotal }}</div>

            <div class="cart-items" v-if="this.$parent.hasItemInCart">
                    <div class="item" :key="index" v-for="(item, index) in cart">
                        <div class="name"><img v-bind:src="item.lesson.image" class="icon">{{ item.lesson.name }}</div>
                        <div class="location">in {{ item.lesson.location }}</div>
                        <div class="price"><span class="grayish">{{ item.quantity }} x</span> £{{ item.lesson.price }}</div>
                        <div class="quantity-selection">
                            <button @click="decreaseQuantity(item)"><i class="fas fa-minus"></i></button><span>{{ item.quantity }}</span><button @click="increaseQuantity(item)"><i class="fas fa-plus"></i></button>
                        </div>
                        <div class="remove-link"><a href="" @click="removeFromCart(index, $event)">remove</a></div>
                    </div>

                    <div class="checkoutform">
                        <h2>Checkout</h2>
                        <form action="" method="POST">

                            <div class="form-field">
                                <label for="name">Name:</label>
                                <input type="text" name="name" placeholder="Enter your name" v-model.trim="order.name.value">
                                <span class="error">{{ this.$parent.isValidName }}</span>
                            </div>
    
                            <div class="form-field">
                                <label for="phone">Phone:</label>
                                <input type="text" name="phone" placeholder="Enter your phone number" v-model.trim="order.phone.value">
                                <span class="error">{{ this.$parent.isValidPhone }}</span>
                            </div>
    
                            <button type="button" @click="this.$parent.checkout" :disabled="!this.$parent.isValidCheckout">Checkout</button>
                        </form>
                    </div>

                </div>
                <div class="cart-is-empty" v-else>
                    <div class="text">Your cart is empty.</div>
                    <button class="outline" @click="displayCart(false)">View classes & activities</button>
                </div>


        </div>
    </div>
</template>

<script>
export default {
    props: ["cart", "order"],
    name: 'Cart',
    data() {
        return {
            
        }
    },
    methods: {
        displayCart(val) {
            this.$emit('displayCart', val);
        },
        increaseQuantity(item) {
            this.$emit('increase', item);
        },
        decreaseQuantity(item) {
            this.$emit('decrease', item);
        },
        removeFromCart(index, event) {
            if(event) event.preventDefault();
            this.$emit('remove', index);
        }
    },
    computed: {
        cartTotal() {
            return this.$parent.cartTotal;
        }
    }
}
</script>