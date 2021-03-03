<template>
    <main>
        <div class="lesson" :key="index" v-for="(lesson, index) in lessons">
            <div class="lesson-wrapper">
                <div class="image" v-bind:style="{ 'background-image': 'url(' + lesson.image + ')' }"></div>
                <div class="info">
                    <div class="name">{{ lesson.name }}</div>
                    <div class="location">in {{ lesson.location }}</div>
                    <div class="avai-spaces">Spaces left: {{ getAvailability(lesson) }}</div>
                    <div class="price">£{{ lesson.price }}</div>
                    <button @click="addToCart(lesson)" :disabled="isAvailable(lesson)">Add to cart</button>
                </div>
            </div>
        </div>
    </main>
</template>

<script>
export default {
    props: ["lessons"],
    name: 'LessonsList',
    methods: {
        addToCart(lesson) {
            this.$emit('add', lesson);
        },
        getAvailability(lesson) {
            return this.$parent.getAvailability(lesson);
        }, 
        isAvailable(lesson) {
            return this.$parent.isAvailable(lesson);
        }
    }
}
</script>

<style scoped>

</style>