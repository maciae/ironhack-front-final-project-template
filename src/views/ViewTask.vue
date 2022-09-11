<template>
  <div>
    <div v-if="dataLoaded">
        <form 
            @submit.prevent="createTask" 
            class="p-8 flex flex-col bg-light-grey rounded-md shadow-lg"
            >
            <h1 class="text-3xl text-at-light-green mb-4">
                Edit task {{ data.title }}
            </h1>
            <div class="flex flex-col mb-2">
                <label for="name" class="mb-1 text-sm text-at-light-green">Name</label>
                <input 
                    type="text"
                    required
                    class="p-2 text-gray-500 focus:outline-none" 
                    id="name"
                    v-model="data.title"
                />
            </div>

            <div class="flex flex-col mb-2">

                <label for="name" class="mb-1 text-sm text-at-light-green">Is completed?</label>
                <p class="onoff">
                    <input type="checkbox"
                    required
                    class="p-2 text-gray-500 focus:outline-none hidden" 
                    id="completed"
                    v-model="data.is_complete"
                    >
                    <label for="completed"></label>
                </p>

            </div>

            <div class="d-flex [&>button]:mr-2">
                <button 
                    type="submit" 
                    @click="update"
                    class="mt-6 py-2 px-6 rounded-sm self-start text-sm text-white bg-at-light-green 
                        duration-200 border-solid border-2 border-transparent hover:border-at-light-green hover:bg-white hover:text-at-light-green"
                    >
                    Update
                </button>
                <button 
                    type="submit" 
                    @click="remove"
                    class="mt-6 py-2 px-6 rounded-sm self-start text-sm text-white bg-red-500
                        duration-200 border-solid border-2 border-transparent hover:border-red-600 hover:bg-white hover:text-red-600"
                    >
                    Delete
                </button>
            </div>
        </form>
    </div>

  </div>
</template>

<script>
import { ref } from "vue";
import { supabase } from '../supabase';
import { useUserStore } from "./../store/user";
import { useTaskStore } from "./../store/task";
import { storeToRefs } from "pinia";
import { useRouter, useRoute } from 'vue-router';



export default {
    name: "view-task",
    setup() {
        const name = ref("");
        const userStore = useUserStore();
        const { user } = storeToRefs(userStore);
        const tasksStore = useTaskStore();

        const router = useRouter();
        const route = useRoute();

        const data = ref(null);
        const dataLoaded = ref(null);
        const taskId = route.params.taskId;

        const getData = async() => {
            try{
                data.value = await tasksStore.fetchTask(taskId, user._object.user.id)
                dataLoaded.value = true;
            } catch (error) {
                console.warn(error.message);
            }
        }
        getData();

        // Update Workout
        const update = async () => {
            try {
                await tasksStore.update(taskId, data.value.title, data.value.is_complete)
                router.push({name: "Home"})
            } catch (error) {
                console.warn(error.message);
            }
        };

        // Update Workout
        const remove = async () => {
            try {
                await tasksStore.delete(taskId, data.value.title, data.value.is_complete)
                router.push({name: "Home"})
            } catch (error) {
                console.warn(error.message);
            }
        };
            
        return { 
            name, 
            dataLoaded, 
            data,
            update, 
            tasksStore,
            remove
        };
    }
}
</script>

<style>

</style>