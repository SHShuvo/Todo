<template>
    <div class="container">
        <div class="row mt-4">
            <div class="col-md-8">
                <div class="card">
                    <div class="card-header text-center">
                        <h4 class="fw-bold">Todo List</h4>
                    </div>

                    <div class="card-body">
                        <div v-for="(todo, index) in allTodos" :key="'todoRow'+index" class="row todo mb-2">
                            <div class="col-md-10">{{index+1}}. {{todo.description}}</div>
                            <div class="col-md-2">
                                <button class="btn btn-danger btn-sm" @click.prevent="deleteTodo(todo.id)">Delete</button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div class="col-md-4">
                <div class="card">
                    <div class="card-header text-center">
                        <h4 class="fw-bold">New Todo</h4>
                    </div>
                    <div class="card-body">
                        <div class="form-group">
                            <label for="description" class="control-label">Description</label>
                            <input type="text" @keydown.enter.prevent="createTodo" v-model="description" 
                                class="form-control form-control-sm" id="description" 
                            placeholder="Enter Todo Descrioption">
                            <span class="text-danger mt-1" v-if="errors['description'] && errors.description[0]">{{ errors.description[0] }}</span>
                            
                        </div>
                        <div class="d-flex justify-content-end mt-3">
                            <button class="btn btn-sm btn-success" @click.prevent="createTodo">Add Todo</button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        data(){
            return{
                description:"",
                allTodos:[],
                errors:[],
            }
        },
        methods:{
            async createTodo(){
                this.errors=[];
                try {
                    const data = await axios.post('/api/create-todo',{
                        description:this.description
                    });
                    if(data.status == 201){
                        toast.fire({
                            icon: 'success',
                            title: 'New Created Successfully'
                        });
                        this.description = '';
                        this.allTodos.push(data.data);
                    }
                } 
                catch (e) {
                    if(e.response.status === 422){
                        this.errors = e.response.data.errors;
                    }
                    toast.fire({
                        icon: 'error',
                        title: e.response.data.msg?e.response.data.msg:'Enter data properly',    
                    })
                }
            },
            async loadTodos(){
                const {data} = await axios.get('/api/all-todos');
                this.allTodos = data;
            },
            async deleteTodo(id){
                const data = await axios.get(`/api/delete-todos/${id}`);
                if(data.status == 200){
                    let index = this.allTodos.findIndex(el=>el.id == id);
                    this.allTodos.splice(index,1);
                    toast.fire({
                        icon: 'success',
                        title: 'Todo Deleted'
                    });
                }
            }
        },
        mounted() {
            this.loadTodos();
        }
    }
</script>
<style scoped>
    .todo{
        border-bottom: 1px solid rgba(0, 0, 0, 0.125);
        padding-bottom: .5rem;
    }
</style>
