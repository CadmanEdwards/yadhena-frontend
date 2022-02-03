<template>
  <v-app>
    <v-main>
      <v-container
                   class="fill-height"
                   >
        <v-row
               justify="center"
               >
          <v-col
                 cols="12"
                 sm="8"
                 md="6"
                 >
            <v-card  class=" elevation-12">
              <v-toolbar flat class="info" dark justify="center">
                <v-row justify="center" class="">
                  <v-col align="center">
                    <h2>Reset Password</h2>
                  </v-col>
                </v-row>
              </v-toolbar>
              <v-row justify="center" class="mt-5">
                <v-col align="center">
                  <NuxtLogo />
                </v-col>
              </v-row>
              <v-card-text>
                <v-form
                        v-if="showForm"
                        method="post"
                        ref="form"
                        lazy-validation
                        >
                  <v-text-field
                                readonly
                                v-model="credentials.email"
                                label="E-mail"
                                :rules="Rules"
                                required
                                >
                  </v-text-field>

                  <v-text-field
                                v-model="credentials.password"
                                label="Password"
                                type="password"
                                :rules="Rules"
                                required
                                >
                  </v-text-field>

                  <v-text-field
                                v-model="credentials.password_confirmation"
                                label="Confirm Password"
                                type="password"
                                :rules="Rules"
                                required
                                >
                  </v-text-field>
               

                  <div class="mb-3" v-for="(error,index) in errors" :key="index">
                      <span  class="error--text ">
                          {{error}}
                      </span>
                  </div>

                   <div class="mb-3" style="width:350px;" v-for="(message,index) in messages" :key="index">
                      <span  class="success--text ">
                          {{message}}
                      </span>
                  </div>

                  <v-row>
                    <v-col>
                  <v-btn  color="primary"  dark :loading="loading"  @click="submit">Submit
                  </v-btn>

                    </v-col>
                    <v-col>
                    <div class="text-right mt-2"><a href="/login">Click here to login</a></div>


                    </v-col>
                  </v-row>
                  
                  

               
                </v-form>

               
               <v-row v-else  justify="center" class="mt-5">
                <v-col align="center">
                  
                    <h3 class="red--text" v-for="(error,index) in errors" :key="index">
                      {{error}}
                    </h3>

                    <div class=" mt-2">
                      <v-btn to="/" class="primary mt-2">

                        Go Home

                      </v-btn>
                      </div>

                </v-col>
              </v-row>

                
              </v-card-text>
              
              <v-card-actions>
              </v-card-actions>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>
<script>
  export default {
    auth : false,
    layout:'guest',
    data:() => ({
      showForm : true,
      messages : [],
      errors : [],
      loading:false,  
      credentials : {
        email:'',
        password : '',
        password_confirmation : '',
        token : ''
      },
      Rules : [ v => !!v || 'This field is required']
    }),
     created () {
      this.initialize()
    },
     
    methods:{

        async initialize () {

         this.credentials.token = this.$route.params.token;

          await this.$axios.get(`/reset_password_link/${this.credentials.token }`).then(res => {

          if(!res.data.status){
             this.errors = res.data.errors;
          }
          else{
            this.errors = [];
            this.credentials.email = res.data.email;            
          }
        });
       

      },
    
    submit () {

      if(this.$refs.form.validate()){   

      this.loading = true

      try {
            this.$axios.post('reset_password_update',this.credentials) 
                .then((res) => {

                  console.log(this.credentials,res);
                  

                    if(res.data.status){
                        this.messages = res.data.message;
                        this.errors = [];
                        this.loading = false;
                    }
                    else{
                      this.messages = [];
                      this.errors = res.data.errors;
                      this.loading = true;
                    }
                })
                .catch((e) => {
                      this.messages = [];
                      this.errors = e.response.data.errors;
                      this.loading = true;
                });
        

      } catch (error) {
                     
      }

      setTimeout(() =>  this.loading = false,3000);
      }
  }
  }
  }
</script>