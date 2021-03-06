<template>
  <div id="app">

    
    
        <nav>
            <div class="container">
                <div>{{ sitename }}</div>
                <button @click="displayCart(true)" class="cart-btn" :disabled="!hasItemInCart">£{{ cartTotal }}<i class="fas fa-shopping-basket"></i></button>
            </div>
        </nav>

        <div class="container filters">

            <div class="filters-container">
                <span class="name">Sort by:</span>
                <div class="filters-selection">
                    <button :key="index" v-for="(filter, index) in filters" @click="selectedFilter = filter" v-bind:class="{ active: isSelectedFilter(filter) }">{{filter}}</button>
                </div>
            </div>
            <div class="filters-container">
                <div class="filters-selection">
                    <button :key="index" v-for="(order, index) in orderBy" @click="selectedOrder = order" v-bind:class="{ active: isSelectedOrder(order) }">{{order}}</button>
                </div>
            </div>
        </div>
        

        <LessonsList :lessons="sortedLessons" @add="addToCart"/>

        <Cart v-bind:class="{ showCart: isCartShown }" 
              :cart="cart"
              :order="order"
              @displayCart="displayCart" 
              @increase="increaseQuantity" 
              @decrease="decreaseQuantity"
              @remove="removeFromCart"/>

        
    
  </div>
</template>

<script>
import Cart from './components/Cart.vue';
import LessonsList from './components/LessonsList.vue';

export default {
  name: 'App',
  components: {
    LessonsList,
    Cart
  },
  data() {
    return {
        sitename: 'After school classes & activities',
        cart: [],
        lessons: [],
        cartShown: false,
        filters: ['name', 'location', 'price', 'availability'],
        orderBy: ['ascending', 'descending'],
        selectedFilter: 'name',
        selectedOrder: 'ascending',
        order: {
            name: {
                value: '',
                error: false,
                errorMessage: ''
            },
            phone: {
                value: '',
                error: false,
                errorMessage: ''
            }
        }
      }
    },
    methods: {
        addToCart(lesson) {
            
                
                let exists = false;
                this.cart.forEach(item => {
                    if(item.lesson == lesson) {
                        exists = true;
                        item.quantity++;
                    }
                });

                if(!exists) {
                    this.cart.push({
                        lesson: lesson,
                        quantity: 1
                    });
                }

        },
        getAvailability(lesson) {
            return lesson.availability - this.countInCart(lesson);
        },
        isAvailable(lesson) {
            return (this.getAvailability(lesson) <= 0) ? true : false;
        },
        countInCart(lesson) {
            let count = 0;
            this.cart.forEach(item => {
                if(item.lesson === lesson) {
                    count = item.quantity;
                }
            });
            return count;
        },
        removeFromCart(index) {
            this.cart.splice(index, 1);
        },
        isSelectedFilter(filter) {
            return filter == this.selectedFilter ? true : false;
        },
        isSelectedOrder(order) {
            return order == this.selectedOrder ? true : false;
        },
        decreaseQuantity(item) {
            if(item.quantity > 1) {
                item.quantity--;
            }
        },
        increaseQuantity(item) {
            if(this.getAvailability(item.lesson) >= 1) {
                item.quantity++;
            }
        },
        addOrder(body) {
            var raw = JSON.stringify(body);

            fetch(`https://coursework2.herokuapp.com/api/orders`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: raw,
            }).catch(err => console.log(err));

        },
        updateLessonAvailability(id, quantity) {

            fetch(`https://coursework2.herokuapp.com/api/lessons/${id}/reduce/availability/${quantity}`, {
                method: 'PUT',
                headers: {
                    'Content-Type': 'application/json'
                }
            }).catch(err => console.log(err));

        },
        checkout() {
            
            this.cart.forEach(item => {

                let body = {
                    "name": this.order.name.value,
                    "phone": this.order.phone.value,
                    "lessonID": item.lesson._id,
                    "quantity": item.quantity
                };
                this.addOrder(body);
                this.updateLessonAvailability(item.lesson._id, item.quantity);

            });

            alert("Thanks for your order");
            this.getLessons();
            this.cart = [];
        },
        displayCart(value) {

            this.cartShown = value;
            if(value) {
                document.body.classList.add('body-no-scroll');
            } else {
                document.body.classList.remove('body-no-scroll');
            }

        },
        getLessons() {
            fetch('https://coursework2.herokuapp.com/api/lessons')
                .then(response => response.json())
                .then(data => this.lessons = data)
                .catch(err => console.log(err));
        }
    },
    computed: {
        cartCount() {
            return this.cart.length;
        },
        hasItemInCart() {
            return (this.cartCount <= 0) ? false : true;
        },
        cartTotal() {
           let total = 0;
           this.cart.forEach(item => {
            total += item.lesson.price * item.quantity;
           });
           return total.toFixed(2);
        },
        sortedLessons() {
            let lessonsArray = this.lessons.slice(0); 
            let that = this;
            function compare(a, b) {
                if (a[that.selectedFilter] > b[that.selectedFilter]) {
                    return that.selectedOrder == 'ascending' ? 1 : -1;
                }
                if (a[that.selectedFilter] < b[that.selectedFilter]) {
                    return that.selectedOrder == 'ascending' ? -1 : 1;
                }
                return 0; 
            }
            return lessonsArray.sort(compare); 
        },
        isCartShown() { 
            return this.cartShown; 
        },
        isValidCheckout() {
            return !this.order.name.error && !this.order.phone.error;
        }, 
        isValidName() {
            let item = this.order.name;

            if(item.value.length <= 0) {
                item.error = true;
                item.errorMessage = 'required';
                return item.errorMessage;
            }

            let test = /^[a-zA-Z ]+$/.test(item.value);

            if(test) {
                item.error = false;
                item.errorMessage = '';
                return item.errorMessage;
            } 

            item.error = true;
            item.errorMessage = 'only letters allowed';
            return item.errorMessage;
            
        }, 
        isValidPhone() {

            let item = this.order.phone;

            if(item.value.length <= 0) {
                item.error = true;
                item.errorMessage = 'required';
                return item.errorMessage;
            }

            let test = /^\d+$/.test(item.value);
            
            if(test) {
                item.error = false;
                item.errorMessage = '';
                return item.errorMessage;
            } 

            item.error = true;
            item.errorMessage = 'only numbers allowed';
            return item.errorMessage;

        }
    },
    mounted() {
        this.getLessons();
    }
}
</script>

<style>


@media only screen and (max-width: 720px) {
    
    body {
        padding-top: 60px;
    }

    .filters {
        display: block;
    }

    .filters-container {
        display: block;
        margin: 0 0 30px 0;
    }

    .filters-container span.name {
        margin: 0 0 20px 0;
    }

    .filters-selection {
        display: block;
    }

    .filters-selection button {
        width: 100%;
        border: 0;
        border-bottom: 1px solid #ddd;
        height: 50px;
    }


    .filters-selection button:last-child {
        border-bottom: 0;
    }

    .filters-selection button:hover {
        color: #42b983;
    }

    .filters-selection button.active {
        color: #fff;
        background: #42b983;
    }  

    nav {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        background: #fff;
        z-index: 2;
        font-size: 14px;
    }


}

</style>
