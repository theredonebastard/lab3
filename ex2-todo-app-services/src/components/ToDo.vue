<template>
  <div class="todo">
    <div class="card text-left">
      <div class="card-header">
        ToDo Component
      </div>
      <div class="card-body">
        <h5 class="card-title">My tasks</h5>
        <div class="card-text">
            <modal v-show="isModalVisible">
            <form>
              <div class="form-group">
                <label for="task-title">Task title</label>
                <input v-model="updatingModel.title" type="text" class="form-control" id="task-title"
                   aria-describedby="emailHelp">
              </div>
              <div class="form-group">
                <label for="task-desc">Task description</label>
                <textarea v-model="updatingModel.description" class="form-control" id="task-desc"></textarea>
              </div>
              <button type="submit" class="btn btn-primary" @click.prevent="updateTask" :disabled="!isValid(updatingModel)">Submit</button>
            </form>
          </modal>

          <form>
            <div class="form-group">
              <label for="task-title">Task title</label>
              <input v-model="model.title" type="text" class="form-control" id="task-title"
                     aria-describedby="emailHelp">
            </div>
            <div class="form-group">
              <label for="task-desc">Task description</label>
              <textarea v-model="model.description" class="form-control" id="task-desc"></textarea>
            </div>
            <button type="submit" class="btn btn-primary" @click.prevent="submit" :disabled="!isValid(model)">Submit</button>
          </form>

          <ul class="pt-3">
            <li v-for="(item, index) in filteredTaskList" :key="index" class="d-flex justify-content-between mb-3">
              <div>
                <h4 v-if="item.status === 'completed'"><s>{{item.title}}</s></h4>
                <h4 v-else>{{item.title}}</h4>
                <p>{{item.description}}</p>
              </div>
              <button class="btn btn-primary" @click="deleteTask(item.id)">Delete</button>
              <button class="btn btn-primary" @click="showModal(item.id)">Update</button>
              <button class="btn btn-primary" @click="updateCompletion(item)">Completed</button>
            </li>
          </ul>
        </div>
        <div class="card-footer">
          <button class="btn" :class="{'btn-primary':!filterStatus}" @click="filterStatus = ''">All</button>
          <button class="btn" :class="{'btn-primary': filterStatus === 'completed'}"
                  @click="filterStatus = 'completed'">Completed
          </button>
          <button class="btn" :class="{'btn-primary': filterStatus === 'incompleted'}"
                  @click="filterStatus = 'incompleted'">InCompleted
          </button>
          Active tasks count: {{ taskList.length }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import {Task} from '../models/task';
  import modal from './modal.vue';

  export default {
    name: "ToDo",
    components: {
      modal,
    },
    props: {},
    data: () => {
      return {
        model: new Task(),
        updatingModel: new Task(),
        taskList: [],
        filterStatus: "",
        loading: false,
        isModalVisible: false,
      }
    },
    async mounted() {
      this.loading = true;
      this.taskList = await this.$services.todo.fetch();
      this.loading = false;
    },
    methods: {
      submit() {
        this.$services.todo.save(this.model).then((res)=>{
          this.taskList.push(res);
          this.model = new Task();
        })
      },
      deleteTask(id){
        this.$services.todo.delete(id).then((res)=>{
          this.taskList = res;
        });
      },
      updateCompletion(item){
        item.status = 'completed';
        this.updatingModel = item;
        this.updateTask();
      },
      updateTask(){
        this.isModalVisible = false;
        this.$services.todo.update(this.updatingModel).then((res)=>{
          this.taskList = res;
          this.updatingModel = new Task();
        });
      },
      async showModal(id) {
        this.isModalVisible = true;
        this.updatingModel = await this.$services.todo.find(id);
      },
      isValid(currentModel) {
        return currentModel.title && currentModel.description;
      }
    },
    watch: {
      message() {
        console.log("Message changed");
      }
    },
    computed: {
      messageLength() {
        return ("" + this.message).length;
      },
      filteredTaskList() {
        return this.taskList.filter((item) => {
          if (!this.filterStatus) return true;
          return item.status === this.filterStatus;
        });
      }
    }
  }
</script>

<style scoped>
  .todo {
    background-color: aqua;
  }
</style>
