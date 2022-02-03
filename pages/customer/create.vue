<template>
<v-app>

<v-snackbar
v-model="snackbar"
:top="'top'"
>
{{msg}}
<v-btn      
text
@click="snackbar = false"
>
Close
</v-btn>
</v-snackbar>

<v-card>
<v-toolbar flat dark>
<v-toolbar-title class="headline">Create</v-toolbar-title>
</v-toolbar>

<v-form ref="form" lazy-validation>
<v-card-text>
<v-container>

<v-row>
<v-col>
<v-text-field :rules="Rules" v-model="contact_person_name" label="Contact Person Name"></v-text-field>
</v-col>
<v-col>
<v-text-field :rules="Rules" v-model="trade_name" label="Trade Name"></v-text-field>
</v-col>
<v-col>
<v-text-field :rules="Rules" v-model="company_name" label="Company Name"></v-text-field>
</v-col>
</v-row>
<v-row>
<v-col>
<v-text-field :rules="Rules"  v-model="email" label="Email"></v-text-field>
<span v-if="errors.email" class="error--text">{{errors.email[0]}}</span>
</v-col> 

    <v-col>
<v-text-field :rules="Rules" type="password" v-model="password" label="Password"></v-text-field>
<span v-if="errors.password" class="error--text">{{errors.password[0]}}</span>
</v-col>

</v-row>
<v-row>
<v-col>
<v-text-field type="number" min="0" v-model="ntn" label="VAT Number"></v-text-field>
</v-col>
<v-col>
<v-select
:rules="Rules"
v-model="customer_category_id" 
:items="customer_categories"
item-value="id"
item-text="customer_category_name" 
label="Group"
></v-select>
</v-col>
</v-row>
<v-row>
<v-col>
<v-text-field type="number" min="0" :rules="Rules" v-model="phone_number" label="Phone Number"></v-text-field>
</v-col>
<v-col>
<v-text-field type="number" min="0" :rules="Rules" v-model="mobile_number" label="Mobile Number"></v-text-field>
</v-col>
</v-row>
<v-row>
<v-col>
<v-text-field :rules="Rules" v-model="address" label="Address"></v-text-field>
</v-col>
</v-row>
<v-row> 
<v-col>
<v-select
:rules="Rules"
v-model="state_id" 
:items="states"
@change="get_cities_by_id"
item-value="id"
item-text="state_name" 
label="Emirate"
></v-select>
</v-col>
<v-col>
<v-select
:rules="Rules"
v-model="city_id" 
:items="cities"
item-value="c_id"
item-text="c_name" 
label="City"
></v-select>

</v-col>
</v-row>
<v-row>
<v-col>
<v-select
:rules="Rules"
v-model="payment_type" 
:items="[{payment_type:'cash'},{payment_type:'credit'}]"
item-value="payment_type"
item-text="payment_type" 
label="Payment Type"
></v-select>

</v-col>
</v-row>
<v-row>
     <v-col>
      <v-menu
        ref="delivery_from"
        v-model="delivery_from_menu"
        :close-on-content-click="false"
        :nudge-right="40"
        :return-value.sync="delivery_from_time"
        transition="scale-transition"
        offset-y
        max-width="290px"
        min-width="290px"
      >
        <template v-slot:activator="{ on }">
          <v-text-field
            v-model="delivery_from_time"
            label="Delivery From"
            readonly
            v-on="on"
          ></v-text-field>
        </template>
        <v-time-picker
          v-if="delivery_from_menu"
          v-model="delivery_from_time"
          full-width
          :max="delivery_to_time"
          @click:minute="$refs.delivery_from.save(delivery_from_time)"
        ></v-time-picker>
      </v-menu>
    </v-col>
 <v-col>
      <v-menu
        ref="delivery_to"
        v-model="delivery_to_menu"
        :close-on-content-click="false"
        :nudge-right="40"
        :return-value.sync="delivery_to_time"
        transition="scale-transition"
        offset-y
        max-width="290px"
        min-width="290px"
      >
        <template v-slot:activator="{ on }">
          <v-text-field
            v-model="delivery_to_time"
            label="Delivery To"
            readonly
            v-on="on"
          ></v-text-field>
        </template>
        <v-time-picker
          v-if="delivery_to_menu"
          v-model="delivery_to_time"
          full-width
          :min="delivery_from_time"
          @click:minute="$refs.delivery_to.save(delivery_to_time)"
        ></v-time-picker>
      </v-menu>
    </v-col>      
</v-row>
<v-row>
<v-col>
<v-checkbox
color="primary"
v-model="IsActive"
label="Is Active"
></v-checkbox>
</v-col>
</v-row>

<v-row>
<v-col>
<v-btn
color="primary"
class="mr-4 accent--text"
@click="register"
>
Submit
</v-btn>
</v-col>
</v-row>

</v-container>
</v-card-text>





</v-form>

<template v-slot:item.action="{ item }">
<v-icon
small
class="mr-2"
@click="editItem(item)"
>
mdi-pencil
</v-icon>
<v-icon
small
@click="deleteItem(item)"
>
mdi-delete
</v-icon>
</template>
<template v-slot:no-data>
<!-- <v-btn color="primary" @click="initialize">Reset</v-btn> -->
</template>
</v-card>
</v-app>
</template>


<script>

export default {
async created () {
const customer_categories = await this.$axios.get('customer_group');
this.customer_categories = customer_categories.data;

const states = await this.$axios.get('state');
this.states = states.data;

},
data:() => ({

delivery_from_menu: false,
delivery_to_menu: false,

delivery_from_time:null,
delivery_to_time:null,

IsActive:'',
customer_categories:[],
states:[],
cities:[],

contact_person_name : "",
trade_name : "",
company_name : "",
payment_type:"",
email : "",
password :"",
ntn : "",
customer_category_id : "",
phone_number : "",
mobile_number : "",
address : "",
state_id : "",
city_id : "",
msg:"",
errors:[
{password:''},
{email:''}
],

snackbar:false,
Rules : [
v => !!v || 'This field is required',
],

}),
computed:{
  trim_me () {
    //return this.phone_number = Math.abs(this.phone_number);
  }
},
methods:{

get_cities_by_id(){
this.$axios.get('state/'+this.state_id).then((res) => this.cities = res.data);
},
register(){
var payload = {  
contact_person_name : this.contact_person_name,
trade_name : this.trade_name,
company_name : this.company_name,
email : this.email,
password : this.password,
ntn : this.ntn,
customer_category_id : this.customer_category_id,
phone_number : this.phone_number,
mobile_number : this.mobile_number,
address : this.address,
state_id : this.state_id,
city_id : this.city_id,
payment_type : this.payment_type,
delivery_from : this.delivery_from_time,
delivery_to : this.delivery_to_time,
IsActive:this.IsActive ? 1 : 0 
};


if(this.$refs.form.validate()){

this.$axios.post('customer',payload).then((res) => {

this.snackbar = res.data.response_status;

if(res.data.response_status){

this.msg = res.data.message;
setTimeout(() => {
  this.$router.push('/customer');
},1000);
}
else{
this.errors = res.data.errors;
}


});

}

},

}

}
</script>