<template>
    <div>
        <draggable 
        v-model="myArray" 
        group="people" 
        @start="drag=true" 
        @end="drag=false" 
        item-key="id">
        <template #item="{element}">
            <div>{{element.name}}</div>
        </template>
        </draggable>
    </div>
</template>

<script>
import draggable from 'vuedraggable'
    export default {
        components: {
            draggable,
        },
        data() {
          return {
            drag: false,
            myArray: [
            { id: 1, name: 'Item 1' },
            { id: 2, name: 'Item 2' },
            { id: 3, name: 'Item 3' },
            { id: 4, name: 'Item 4' },
            { id: 5, name: 'Item 5' }
            ]
          }
        },
    }
</script>

<style lang="scss" scoped>

</style>