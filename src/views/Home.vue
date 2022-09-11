<template>
  <div>
    <div>
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
    <div v-if="dataLoaded && dataCompleted.length === 0 && dataPending.length === 0" class="container mt-10 px-4">
        <!-- NO DATA -->
        <div class="w-full flex flex-col items-center">
            <h1 class="text-2xl">Looks empty here...</h1>
        </div>
    </div>
    <!-- <div v-else class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6">
        <router-link 
            class="flex flex-col items-center bg-light-grey p-8 shadow-md cursor-pointer"
            :to="{ name: ''}"
            v-for="(task, index) in data"
            :key="index"
        >

        <h1>{{task.title}}</h1>
        </router-link>
    </div>
    -->
    <div v-else-if="dataLoaded" class="grid gap-x-8 gap-y-4 grid-cols-4 m-b-5">
        <div class="col-span-2">
            <h2 class="text-3xl mb-5 mt-8 text-center font-bold text-sky-700 border-b-4 border-sky-800">
                To-Do Tasks
            </h2>
            <div 
                class="to-do-list [&>a]:bg-sky-200 [&>a]:border-2 [&>a]:border-slate-900 bg-sky-100 h-full p-5"
                @drop="onDrop($event, 1)"
                @dragenter.prevent
                @dragover.prevent
                >
                <router-link 
                    class="flex flex-col items-center mb-5 bg-light-grey p-5 shadow-md cursor-pointer"
                    :to="{ name: 'ViewTask', params: { taskId: task.id } }"
                    v-for="(task, index) in dataPending"
                    v-bind:id="task.id"
                    :key="index"
                    draggable="true"
                    @dragstart="startDrag($event, task, 1)"
                >
                <h1>{{task.title}}</h1>
                </router-link>
            </div>
        </div>
        <div class="col-span-2">
            <h2 class="text-3xl mb-5 mt-8 text-center font-bold text-lime-700 border-b-4 border-lime-800">
                Done Tasks
            </h2>
            <div 
                class="done-list [&>a]:bg-lime-200 [&>a]:border-2 [&>a]:border-slate-900  bg-lime-100 h-full p-5"
                @drop="onDrop($event, 2)"
                @dragenter.prevent
                @dragover.prevent
                >
                <router-link
                    class="flex flex-col items-center mb-5 bg-light-grey p-5 shadow-md cursor-pointer"
                    :to="{ name: 'ViewTask', params: { taskId: task.id } }"
                    v-for="(task, index) in dataCompleted"
                    v-bind:id="task.id"
                    :key="index"
                    draggable="true"
                    @dragstart="startDrag($event, task, 2)"
                >
                <h1>{{task.title}}</h1>
                </router-link>
            </div>
        </div>
    </div>
  </div>
</template>

<script>
import { ref } from "vue";
import { useUserStore } from "./../store/user";
import { storeToRefs } from "pinia";
import { useTaskStore } from "./../store/task";

export default {
    name: "home",
    setup() {
        const dataCompleted = ref([]);
        const dataPending = ref([]);
        const dataLoaded = ref(null);
        const tasksStore = useTaskStore();
        const userStore = useUserStore();
        const { user } = storeToRefs(userStore);
        const name = ref("");

        const createTask = async () => {
            if (name.value.length > 0){    
                try {
                    console.log("name value es " + name.value)
                    await tasksStore.create(name.value, user._object.user.id);
                    getData();
                } catch (error) {
                    console.warn(error.message);
                }
            } else {
              console.warn("Error: El nombre de la tarea no puede estar vacío");
            }
        }

        const getData = async () => {
            console.log("getadata");
            try {
                dataCompleted.value = await tasksStore.fetchTasks(user._object.user.id, true);
                dataPending.value = await tasksStore.fetchTasks(user._object.user.id, false);
                dataLoaded.value = true;
            }
            catch (error) {
                console.warn(error.message);
            }
        };
        getData();
        const startDrag = (event, item, originList) => {
            console.log(item);
            event.target.style.opacity=1;
            event.dataTransfer.dropEffect = "move";
            event.dataTransfer.effectAllowed = "move";
            event.dataTransfer.setData("itemID", item.id);
            event.dataTransfer.setData("originList", originList);
        };
        const onDrop = (event, list) => {
            const itemID = event.dataTransfer.getData("itemID");
            const originList = event.dataTransfer.getData("originList");
            let isCompleted = true;
            if (originList != list) {
                if (list == 1) { // TODO LIST
                    isCompleted = false;
                }
                const updateCompleted = async () => {
                    try {
                        await tasksStore.setTaskCompleted(itemID, isCompleted);
                        getData();
                    }
                    catch (error) {
                        console.warn(error.message);
                    }
                };
                updateCompleted();
            }

        };
        return {
            dataCompleted,
            dataPending,
            dataLoaded,
            startDrag,
            onDrop,
            name,
            createTask
        };
    }
}
</script>

<style>

</style>