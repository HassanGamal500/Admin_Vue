
<style scoped>
.widget-user-header{
    /* background: url('admin-lte/dist/img/photo1.png') center center; */
    background-position: center;
    background-size: cover;
    height: 250px;
}
</style>


<template>
  <div class="container">
    <div class="row justify-content-center">
      <div class="col-md-12 mt-3">
        <div class="card card-widget widget-user">
            <!-- Add the bg color to the header using any of the bg-* classes -->
            <div class="widget-user-header text-white"
                style="background: url('admin-lte/dist/img/photo1.png');">
                <h3 class="widget-user-username text-right">Elizabeth Pierce</h3>
                <h5 class="widget-user-desc text-right">Web Designer</h5>
            </div>
            <div class="widget-user-image">
                <img class="img-circle" :src="getProfilePhoto()" alt="User Avatar" />
            </div>
            <div class="card-footer">
                <div class="row">
                <div class="col-sm-4 border-right">
                    <div class="description-block">
                    <h5 class="description-header">3,200</h5>
                    <span class="description-text">SALES</span>
                    </div>
                    <!-- /.description-block -->
                </div>
                <!-- /.col -->
                <div class="col-sm-4 border-right">
                    <div class="description-block">
                    <h5 class="description-header">13,000</h5>
                    <span class="description-text">FOLLOWERS</span>
                    </div>
                    <!-- /.description-block -->
                </div>
                <!-- /.col -->
                <div class="col-sm-4">
                    <div class="description-block">
                    <h5 class="description-header">35</h5>
                    <span class="description-text">PRODUCTS</span>
                    </div>
                    <!-- /.description-block -->
                </div>
                <!-- /.col -->
                </div>
                <!-- /.row -->
            </div>
        </div>

        <div class="card">
            <div class="card-header p-2">
                <ul class="nav nav-pills">
                    <li class="nav-item"><a class="nav-link active" href="#activity" data-toggle="tab">Activity</a></li>
                    <li class="nav-item"><a class="nav-link" href="#settings" data-toggle="tab">Settings</a></li>
                </ul>
            </div><!-- /.card-header -->
            <div class="card-body">
                <div class="tab-content">
                    <div class="tab-pane" id="activity">
                        <h3 class="text-center">Display User Activity</h3>
                    </div>
                    <!-- /.tab-pane -->

                    <div class="tab-pane active" id="settings">
                        <form class="form-horizontal">
                            <div class="form-group row">
                                <label for="inputName" class="col-sm-2 col-form-label">Name</label>
                                <div class="col-sm-10">
                                <input type="text" v-model="form.name" class="form-control" id="inputName" placeholder="Name"  :class="{ 'is-invalid': form.errors.has('name') }">
                                <has-error :form="form" field="name"></has-error>
                                </div>
                            </div>
                            <div class="form-group row">
                                <label for="inputEmail" class="col-sm-2 col-form-label">Email</label>
                                <div class="col-sm-10">
                                <input type="email" v-model="form.email" class="form-control" id="inputEmail" placeholder="Email"  :class="{ 'is-invalid': form.errors.has('email') }">
                                <has-error :form="form" field="email"></has-error>
                                </div>
                            </div>
                            <div class="form-group row">
                                <label for="photo" class="col-sm-2 col-form-label">Photo</label>
                                <div class="col-sm-10">
                                <input type="file" @change="updateProfile" class="form-input" id="photo" placeholder="Photo"  :class="{ 'is-invalid': form.errors.has('photo') }">
                                <has-error :form="form" field="photo"></has-error>
                                </div>
                                <img :src="imageSrc" width="100px">
                            </div>
                            <div class="form-group row">
                                <label for="inputPassword" class="col-sm-2 col-form-label">Password</label>
                                <div class="col-sm-10">
                                <input type="password" v-model="form.password" class="form-control" id="inputPassword" placeholder="Password"  :class="{ 'is-invalid': form.errors.has('password') }">
                                </div>
                                <has-error :form="form" field="password"></has-error>
                            </div>
                            <div class="form-group row">
                                <div class="offset-sm-2 col-sm-10">
                                <button @click.prevent="updateInfo" type="submit" class="btn btn-danger">Submit</button>
                                </div>
                            </div>
                        </form>
                    </div>
                    <!-- /.tab-pane -->
                </div>
                <!-- /.tab-content -->
              </div><!-- /.card-body -->
            </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
    data(){
        return {
            images: {
                // cover: ~admin-lte/dist/img/photo1.png,
                sample: '/admin-lte/dist/img/default-150x150.png'
            },
            form: new Form({
                id: '',
                name: '',
                email: '',
                password: '',
                photo: ''
            }),
            imageSrc: null
        }
    },
    methods: {
        loadProfile(){
            axios.get('api/profile')
            .then(({data}) => {
                this.form.fill(data)
                // console.log(data)
            })
        },
        getProfilePhoto(){
            let photo = (this.form.photo.length > 100) ? this.form.photo :  'img/profile/'+this.form.photo
            return photo
        },
        updateProfile(e){
            // console.log('uploading')
            let file = e.target.files[0]
            let reader = new FileReader()
            // console.log(file)
            if(file['size'] < 2000000){
                reader.onloadend = (file) => {
                    // console.log('Result', reader.result)
                    this.form.photo = reader.result
                    this.imageSrc = reader.result 
                }
            } else {
                Swal.fire({
                    icon: 'error',
                    title: 'Oops...',
                    text: 'You Are Uploading a Large File!'
                })
            }
            
            // reader.onload = function(e) {
            //     this.imageSrc = reader.result            
            // }
            // console.log(file)
            reader.readAsDataURL(file)
        },
        updateInfo(){
            this.$Progress.start()
            this.form.put('api/profile')
            .then(() => {
                Fire.$emit('AfterCreated')
                this.$Progress.finish()
            })
            .catch(() => {
                this.$Progress.fail()
            })
        }
    },
    mounted() {
        // console.log(this.images.sample);
    },
    created(){
        this.loadProfile()
        Fire.$on('AfterCreated', () => {
            this.loadProfile()
        })
    }
};
</script>
