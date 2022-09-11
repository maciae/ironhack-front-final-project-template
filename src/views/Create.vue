<template>
  <div>
    <div v-if="errorMsg" class="mb-10 p-4 rounded-md bg-light-grey shadow-lg">
        <p class="text-red-500">
            {{ errorMsg }}
        </p>
    </div>

    <div v-if="successMsg" class="mb-10 p-4 rounded-md bg-light-grey shadow-lg">
        <p class="text-red-500">
            {{ successMsg }}
        </p>
    </div>

    <form 
        @submit.prevent="createTask" 
        class="p-8 flex flex-col bg-light-grey rounded-md shadow-lg"
        >
        <h1 class="text-3xl text-at-light-green mb-4">
            Create a new ToDo Task
        </h1>
        <div class="flex flex-col mb-2">
            <label for="name" class="mb-1 text-sm text-at-light-green">Name</label>
            <input 
                type="text"
                required
                class="p-2 text-gray-500 focus:outline-none" 
                id="name"
                v-model="name"
            />
        </div>

        <button 
            type="submit" 
            class="mt-6 py-2 px-6 rounded-sm self-start text-sm text-white bg-at-light-green 
                duration-200 border-solid border-2 border-transparent hover:border-at-light-green hover:bg-white hover:text-at-light-green"
            >
            Create
        </button>
    </form>
    
  </div>
</template>

<script>
import { ref } from "vue";
import { supabase } from '../supabase';
import { useUserStore } from "./../store/user";
import { useTaskStore } from "./../store/task";
import { storeToRefs } from "pinia";



export default {
    name: "create",
    setup() {
        const name = ref("");
        const errorMsg = ref(null);
        const successMsg = ref(null);
        const userStore = useUserStore();
        const taskStore = useTaskStore();
        const { user } = storeToRefs(userStore);

        const createTask = async () => {
            if (name.value.length > 0){    
                try {
                    console.log("name value es " + name.value)
                    await taskStore.create(name.value, user._object.user.id);
                    successMsg.value = `Success: Task with name ${ name.value } created!`;
                } catch (error) {
                    errorMsg.value = `Error: ${error.mesage}`
                }
            } else {
                errorMsg.value = "Error: El nombre de la tarea no puede estar vacío"
            }
        }
            
        return { name, createTask, errorMsg, successMsg };
    }
}
</script>

<style>

</style>