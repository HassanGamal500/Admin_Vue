<template>
    <div class="container">
        <div class="row">
          <div class="col-md-12">
            <div class="card mt-5">
              <div class="card-header">
                <h3 class="card-title">Users Table</h3>

                <div class="card-tools">
                    <button class="btn btn-success" @click="newModel">
                        Add New 
                        <i class="fas fa-user-plus fa-fw"></i>
                    </button>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover text-nowrap">
                  <thead>
                    <tr>
                      <th>ID</th>
                      <th>Name</th>
                      <th>Email</th>
                      <th>History</th>
                      <th>Modify</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="user in users" :key="user.id">
                      <td>{{ user.id }}</td>
                      <td>{{ user.name | upText }}</td>
                      <td>{{ user.email }}</td>
                      <td>{{ user.created_at | myDate }}</td>
                      <td>
                          <a href="#" @click="editModel(user)">
                              <i class="fa fa-edit blue"></i>
                          </a>
                          /
                          <a href="#" @click="deleteUser(user.id)">
                              <i class="fa fa-trash red"></i>
                          </a>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
              <!-- /.card-body -->
            </div>
            <!-- /.card -->
          </div>
        </div>
        <!-- Modal Add New User-->
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNewLabel" aria-hidden="true">
        <div class="modal-dialog modal-dialog-centered" role="document">
            <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title" v-show="!editmode" id="addNewLabel">Add New</h5>
                <h5 class="modal-title" v-show="editmode" id="addNewLabel">Update User's Info</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                <span aria-hidden="true">&times;</span>
                </button>
            </div>
            <form @submit.prevent="editmode ? updateUser() : createUser()">
                <div class="modal-body">
                    <div class="form-group">
                        <label>Name</label>
                        <input v-model="form.name" type="text" name="name"
                            placeholder="Name"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                        <has-error :form="form" field="name"></has-error>
                    </div>
                    <div class="form-group">
                        <label>Email</label>
                        <input v-model="form.email" type="email" name="email"
                            placeholder="Email Address"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                        <has-error :form="form" field="email"></has-error>
                    </div>
                    <div class="form-group">
                        <label>Password</label>
                        <input v-model="form.password" type="password" name="password"
                            placeholder="password"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                        <has-error :form="form" field="password"></has-error>
                    </div>
                </div>

                <div class="modal-footer">
                    <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                    <button v-show="editmode" type="submit" class="btn btn-success">Update</button>
                    <button v-show="!editmode" type="submit" class="btn btn-primary">Create</button>
                </div>
                
            </form>
            
            </div>
        </div>
        </div>
    </div>
</template>

<script>
    export default {
        data(){
            return {
                editmode: false,
                users: [],
                form: new Form({
                    id: '',
                    name: '',
                    email: '',
                    password: ''
                })
            }
        },
        methods: {
            newModel(){
                this.editmode = false
                this.form.reset()
                this.form.clear()
                $('#addNew').modal('show')
            },
            editModel(user){
                this.editmode = true
                this.form.reset()
                this.form.clear()
                this.form.fill(user)
                $('#addNew').modal('show')
            },
            loadUsers(){
                axios.get('api/user').then(({data}) => {
                    this.users = data.data
                    // console.log(this.users)
                })
            },
            createUser () {
                this.$Progress.start()
                
                this.form.post('api/user')
                .then(() => {
                    Fire.$emit('AfterCreated')
                    $('#addNew').modal('hide')
                    Toast.fire({
                        icon: 'success',
                        title: 'User Created in successfully'
                    })
                    this.$Progress.finish()
                })
                .catch(() => {

                })
                
            },
            updateUser(){
                this.$Progress.start()
                this.form.put('api/user/'+this.form.id)
                .then(() => {
                    $('#addNew').modal('hide')
                    Swal.fire(
                        'Updated!',
                        'Information has been Updated.',
                        'success'
                    )
                    Fire.$emit('AfterCreated')
                    this.$Progress.finish()
                })
                .catch(() => {
                    this.$Progress.fail()
                })
            },
            deleteUser(id){
                console.log(id)
                Swal.fire({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    icon: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                    }).then((result) => {
                        // Send Request To The Server
                        this.form.delete('api/user/'+id)
                        .then(() => {
                            if (result.value) {
                                Swal.fire(
                                    'Deleted!',
                                    'Your file has been deleted.',
                                    'success'
                                )
                                Fire.$emit('AfterCreated')
                            }
                        })
                        .catch(() => {
                            Swal.fire({
                                icon: 'error',
                                title: 'Oops...',
                                text: 'Something went wrong!'
                            })
                        })

                    })
            }
        },
        created() {
            Fire.$on('searching', () => {
                let query = this.$parent.search
                axios.get('api/findUser?q='+query)
                .then((data) => {
                    this.users = data.data
                    // console.log(data)
                })
                .catch(() => {

                })
            })

            this.loadUsers()
            Fire.$on('AfterCreated', () => {
                this.loadUsers()
            })
            // setInterval(() => this.loadUsers(), 3000)
        }
    }
</script>
