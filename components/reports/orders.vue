<template>
<v-app>
<div class="text-center ma-2">

<v-snackbar
v-model="snackbar"
:top="'top'"
>
{{response.msg}}
<v-btn      
text
@click="snackbar = false"
>
Close
</v-btn>
</v-snackbar>
</div>


<v-data-table
:headers="headers"
:items="orders"
class="elevation-1"
:options.sync="options"
:server-items-length="data.total"
@pagination="paginate"
:items-per-page=10
:footer-props="{
itemsPerPageOptions:[10]
}"
>


<template  v-slot:item.status="{ item }">
<v-chip small class="white--text" :class="myBtnClass(item.status)">
{{item.status}}
</v-chip>
</template>


<template v-slot:top>
<v-toolbar class="primary accent--text title" flat>
Orders Report
<v-spacer></v-spacer>
   <VueJsonToCsv
    :json-data="orders"
    :labels="{ 
      id:{ title: 'order id' },
      order_total:{ title: 'order total' }, 
      order_tax:{ title: 'order tax' }, 
      discounted_price:{ title: 'discounted price' }, 
      order_gross:{ title: 'order gross' }, 
      order_confirmed_date:{ title: 'order confirmed date' }, 
      order_shipped_date:{ title: 'order shipped date' }, 
      order_delivered_date:{ title: 'order delivered date' },  
      delivery_date:{ title: 'delivery date' },  
      payment_due_date:{ title: 'payment due date' },  
      status:{ title: 'status' },  
      email	:{ title: 'email' },  
      phone_number:{ title: 'phone number' },  
      mobile_number:{ title: 'mobile number' },  
      ntn:{ title: 'ntn' },  
      address:{ title: 'address' },  
      company_name:{ title: 'company name' },  
      contact_person_name:{ title: 'contact person name' },  
      payment_type:{ title: 'payment type' },  
      customer_category_name:{ title: 'customer category name' },
      state_name:{ title: 'state name' },
      city_name:{ title: 'city name' },  
      }"    
    >
    <v-btn class="primary mx-1 accent--text no_print">
    <v-icon>mdi-file-export</v-icon><b>&nbsp;Export CSV </b>
    </v-btn>
    </VueJsonToCsv>
</v-toolbar>
<v-row class="px-5"> 
<v-col>
<v-select
v-model="product_id" 
:items="product_list"
item-value="id"
item-text="product_title" 
label="Products"
></v-select>
</v-col>
<v-col>
<v-select
v-model="customer_id" 
:items="customers"
item-value="id"
item-text="company_name" 
label="Customer"
></v-select>
</v-col>
<v-col>
<v-select
v-model="driver_id" 
:items="driver_listing"
item-value="id"
item-text="name" 
label="Driver"
></v-select>
</v-col>
<v-col>
<v-select
@change="get_cities_by_id"
v-model="state_id" 
:items="states"
item-value="id"
item-text="state_name" 
label="State"
></v-select>
</v-col>

<v-col>
<v-select
v-model="city_id" 
:items="cities"
item-value="id"
item-text="city_name" 
label="City"
></v-select>
</v-col>
</v-row>



<v-row class="px-4">
      <v-col>
      <v-menu
      
        v-model="menu_from"
        :close-on-content-click="false"
        :nudge-right="40"
        transition="scale-transition"
        offset-y
        min-width="290px"
      >
        <template v-slot:activator="{ on }">
          <v-text-field
            dense
            v-model="date_from"
            label="From"
            readonly
            v-on="on"
          ></v-text-field>
        </template>
        <v-date-picker color="primary accent--text" v-model="date_from" @input="menu_from = false"></v-date-picker>
      </v-menu>
    </v-col>
   <v-col>
      <v-menu
      
        v-model="menu_to"
        :close-on-content-click="false"
        :nudge-right="40"
        transition="scale-transition"
        offset-y
        min-width="290px"
      >
        <template v-slot:activator="{ on }">
          <v-text-field
            dense
            label="To"
            v-model="date_to"
            readonly
            v-on="on"
          ></v-text-field>
        </template>
        <v-date-picker color="primary accent--text" v-model="date_to" @input="menu_to = false"></v-date-picker>
      </v-menu>
    </v-col>
</v-row>
<v-toolbar style="margin-top:-10px;" flat>

<v-radio-group v-model="status_id" row>
<span>Status</span>&nbsp;&nbsp;
<v-radio color="primary accent--text" label="All" value=""></v-radio>
<v-radio color="primary accent--text"  v-for="status in statusses" :key="status.id" :label="status.status | capitalize " :value="status.id"></v-radio>
</v-radio-group>

</v-toolbar>       
<v-toolbar style="margin-top:-15px;" flat>

<v-radio-group v-model="timestamp" row>
<span>Timestamp</span>&nbsp;&nbsp;
<v-radio color="primary accent--text" label="All" value=""></v-radio>
<v-radio color="primary accent--text" label="today" value="today"></v-radio>
<v-radio color="primary accent--text" label="week" value="week"></v-radio>
<v-radio color="primary accent--text" label="month" value="month"></v-radio>
<v-radio color="primary accent--text" label="year" value="year"></v-radio>
</v-radio-group>

</v-toolbar> 
<v-row class="px-4" style="margin-top:-15px;">
<v-col>
<v-btn @click="filter_records"  class="primary accent--text">
<v-icon>mdi-filter</v-icon> 
Filter
</v-btn>
<v-btn @click="reset"  class="accent primary--text mx-2">
<v-icon>mdi-backup-restore</v-icon>&nbsp;Reset
</v-btn>
</v-col>
</v-row>

  <v-row class="px-4" style="">
  <v-col>
  <v-alert dense class="primary accent--text">
        Total Records: <strong v-if="orders.length > 0">{{orders.length}}</strong> 
  </v-alert>
  </v-col>
  </v-row>
<v-toolbar flat>



<v-dialog v-model="dialog" max-width="1200px">
<!-- <template v-slot:activator="{ on }">
<v-btn color="primary" to="order/create" class="accent--text mb-2">New Item</v-btn>
</template> -->
<v-card>
<!-- <v-card-title>

<span class="headline">{{ formTitle }}</span> -->


<v-card-text>

<v-container>
<v-row>
<v-col>
<v-toolbar
class="primary mb-2 accent--text"
dark
flat
>
<v-toolbar-title>{{ editedItem.company_name }}</v-toolbar-title>
</v-toolbar>
</v-col>


</v-row>


<v-row>
<v-col> 
<template>
<v-simple-table dense>
<template v-slot:default>
<thead>
<tr>
<th class="text-left">Id</th>
<th class="text-left">Item Code</th>
<th class="text-left">Title</th>
<th class="text-left">Image</th>
<th class="text-center">Quantity</th>
<th class="text-left">Unit Price</th>
<th class="text-left">Price</th>
<th class="text-left" v-if="!isReadOnly">Change Product</th>
<th class="text-left" v-if="!isReadOnly">Change Quantity</th>
<th v-if="!isReadOnly">Action</th>
</tr>
</thead>
<tbody>

<tr v-for="(item,index) in editedItem.products" :key="index">

<td>{{ item.id }}</td>
<td>{{ item.legacy_code_sku }}</td>
<td>{{ item.product_title }}</td>
<td height="25" width="15%"><img style="padding-top:5px;" width="30%" :src="item.product_image" alt="Orange Room Digital"/></td>
<td width="14%" class="text-center">{{ item.product_quantity }} 
<span 

:class="get_stock_info(item.stock).class"
>
{{get_stock_info(item.stock).text}}
{{item.stock > 0 ? '(' + item.stock + ')' : '' }} 

</span>  
<!-- v-if="editedItem.order_status_id == 1"  -->
</td>

<td>AED {{ item.product_price }}</td>
<!-- <td>AED {{ item.product_quantity * item.product_price }}</td> -->

<td>AED {{sum_of_product_price(item.product_quantity , item.product_price)}}</td>

<td v-if="!isReadOnly">

<v-select
class="mb-2"
v-model="order_item.id[index]"

:items="order_items"
item-value="id"
item-text="product_title" 
label="Products"
></v-select>

</td>
<td v-if="!isReadOnly"><v-text-field  type="number"  v-model="order_item_quantity[index]"></v-text-field>
</td>
<td v-if="!isReadOnly">
<v-btn class="primary accent--text" @click="get_product(index,item.order_item_id)" fab x-small dark>
<v-icon>mdi-content-save</v-icon>
</v-btn>

</td>

</tr>



</tbody>

</template>
</v-simple-table>
</template>
</v-col>

</v-row>
<v-divider></v-divider>
<v-form ref="form" lazy-validation>

<v-row>
<v-col cols="3">
<v-autocomplete

class="mb-2"
:rules="Rules"
v-if="!isReadOnly"
v-model="editedItem.driver_id" 
:items="drivers"
item-value="id"
item-text="name" 
label="Driver"
></v-autocomplete>
</v-col>
<v-col md="6">
<template>
  <v-row>
    <v-col>
      <v-menu
      
        ref="menu"
        v-model="menu"
        :close-on-content-click="false"
        :return-value.sync="date"
        transition="scale-transition"
        offset-y
        min-width="290px"
      >
        <template v-slot:activator="{ on }">
          <v-text-field
            dense
            v-model="date"
            label="Payment Due Date"
            readonly
            v-on="on"
          ></v-text-field>
        </template>
        <v-date-picker v-model="date" no-title scrollable>
          <v-spacer></v-spacer>
          <v-btn text color="primary" @click="menu = false">Cancel</v-btn>
          <v-btn text color="primary" @click="$refs.menu.save(date)">OK</v-btn>
        </v-date-picker>
      </v-menu>
    </v-col>
   <v-col>
      <v-menu
      
        ref="menu2"
        v-model="menu2"
        :close-on-content-click="false"
        :return-value.sync="date2"
        transition="scale-transition"
        offset-y
        min-width="290px"
      >
        <template v-slot:activator="{ on }">
          <v-text-field
            dense
            v-model="date2"
            label="Delivery Date"
            readonly
            v-on="on"
          ></v-text-field>
        </template>
        <v-date-picker v-model="date2" no-title scrollable>
          <v-spacer></v-spacer>
          <v-btn text color="primary" @click="menu2 = false">Cancel</v-btn>
          <v-btn text color="primary" @click="$refs.menu2.save(date)">OK</v-btn>
        </v-date-picker>
      </v-menu>
    </v-col>
  
    </v-row>

</template>
</v-col>
<v-simple-table dense>
<tbody>
<tr>
<th style="padding-left:75px; width:60%;">Gross Amount</th>
<td style="border:none;">AED {{ get_decimal_value(editedItem.order_gross) }}</td>
</tr>    
<tr>
<th style="padding-left:75px; width:60%;">Tax (VAT %5)</th>
<td style="border:;">AED {{ get_decimal_value(editedItem.order_tax) }}</td>
</tr>
<tr>
<th style="padding-left:75px; width:60%;">Grand Total</th>
<th>AED {{get_decimal_value(editedItem.order_total)}}  </th>
</tr>


</tbody>
</v-simple-table>

</v-row>
<v-row>
<v-col>
<v-btn class="primary accent--text" text @click="close">Cancel</v-btn>
&nbsp;
<v-btn v-if="!isReadOnly" class="primary accent--text" text @click="save">Save</v-btn>
</v-col>

</v-row>

</v-form>



</v-container>
</v-card-text>

</v-card>
</v-dialog>


</v-toolbar>
</template>
<template v-slot:item.indicator="{ item }">

<!-- <v-icon small  v-if="item.order_status_id == 1 && 
(item.products[0].stock == 0 || 
item.products[0].stock == 'Stock does not exist')" 
class="error"
>mdi-checkbox-blank-circle</v-icon> -->

</template>
<template v-slot:item.order_total="{ item }">
{{get_decimal_value(item.order_total)}}  
</template>

<!-- <template v-slot:item.action="{ item }">
<v-icon
small
class="mr-2"
@click="viewItem(item)"
>
mdi-eye
</v-icon>
<v-icon
v-if="item.order_status_id == 1 || item.order_status_id == 2"
small
class="mr-2"
@click="editItem(item)"
>
mdi-pencil
</v-icon>
{{item}}

<v-icon
small
@click="deleteItem(item)"
>
mdi-delete
</v-icon>
</template> -->
</v-data-table>
</v-app>
</template>
<script>
import VueJsonToCsv from '../../node_modules/vue-json-to-csv'
export default {

  components : {
    VueJsonToCsv
  },

data: () => ({
date_from: '',
menu_from: false,
date_to: '',
menu_to: false,  

date: new Date().toISOString().substr(0, 10),
date2: new Date().toISOString().substr(0, 10),
menu: false,
menu2: false,
product_id:'',    
timestamp:'',
status_id:'',
customer_id:'',
driver_id:'',
state_id:'',
city_id:'',

order_item:{
id:[]
},
order_item_quantity:[],
options:{
sortBy:['id','order_total','created_at'],
sortDesc:[true]
},
snackbar:false,
action:'',
search:'',
dialog: false,
headers: [


{
text: 'Order #',
align: 'left',
value: 'id',
},
{
text: 'Company Name',
align: 'left',
sortable: false,
value: 'company_name',
},
{
text: 'Contact Person Name',
align: 'left',
sortable: false,
value: 'contact_person_name',
},
{
text: 'Mobile Number',
align: 'left',
sortable: false,
value: 'mobile_number',
},

{
text: 'Customer Class',
align: 'left',
sortable: false,
value: 'customer_category_name',   
},
{
text: 'Order Total',
align: 'left',
value: 'order_total',
},

{
text: 'Order Status',
align: 'left',
sortable: false,
value: 'status',
},
{
text: 'Order DateTime',
align: 'left',
value: 'created_at',
},
// { text: 'Actions', value: 'action', sortable: false },
],
emailRules: [
v => /.+@.+/.test(v) || 'E-mail must be valid',
],
msg:"",
product_list:[],
snackbar:false,
Rules : [
v => !!v || 'This field is required',
],

drivers:[],
driver_listing:[],
order_items:[],
editedIndex: -1,
editedItem: {
products:[],
driver_id:'',
product_id:'',

},
defaultItem: {
products:[],
driver_id:'',
product_id:'',
},
response : {
msg:''
},
orders:[],
states:[],
cities:[],
customers:[],
statusses:[],
data:[],

change_product:[],



}),

computed: {

isReadOnly(){
return this.action == 'View Item'
},
formTitle () {
return (this.action) ? this.action :this.editedIndex === -1 ? 'New Item' : 'Edit Item'
},

},

watch: {
dialog (val) {
val || this.close()
},
},

created () {
this.initialize()
this.myBtnClass()
this.get_data()
},  
filters: {
  capitalize: function (value) {
    if (!value) return ''
    value = value.toString()
    return value.charAt(0).toUpperCase() + value.slice(1)
  }
},

methods: {
get_cities_by_id(){
var arr = [];
var all = [];


this.$axios.get('state/'+this.state_id).then((res) =>{
  res.data.map(r => arr.push( {id:r.c_id,city_name:r.c_name} ));
  all = [{id:'',city_name:'All'}];
  arr.unshift(all[0]);
  this.cities = arr;
});

},
  reset () {
  this.date_from = '';
  this.date_to = '';
  this.product_id = '';
  this.timestamp = '';
  this.driver_id = '';
  this.status_id = '';
  this.customer_id = '';
  this.state_id = '';
  this.city_id = '';
  this.filter_records()
},

filter_records () {

this.$axios.get('order',{params:{
'per_page':10,
'sort_by':'desc',
'order_by':'id',
'timestamp' : this.timestamp ,
'driver' : this.driver_id,
'status' : this.status_id,
'customer' : this.customer_id,
'state' : this.state_id,
'city' : this.city_id,
'product' : this.product_id,
'from' : this.date_from,
'to' : this.date_to
}})
.then(res => {
this.orders = res.data.orders;
this.data = res.data;
});

},


async get_data () {
var all = [];


const fl = await this.$axios.get('filter_listing');    

all = [{id:'',company_name:'All'}];
fl.data.customers.unshift(all[0]);

all = [{id:'',name:'All'}];
fl.data.drivers.unshift(all[0]);


all = [{id:'',state_name:'All'}];
fl.data.states.unshift(all[0]);


all = [{id:'',city_name:'All'}];
fl.data.cities.unshift(all[0]);


all = [{id:'',product_title:'All'}];
fl.data.products.unshift(all[0]);


var statusses = fl.data.status.map(v=> ({
  id : v.id,
  status : (v.status == 'on the way') ? 'Dispatch' : v.status  
}))



this.statusses = statusses;
this.customers = fl.data.customers;
this.driver_listing = fl.data.drivers;
this.states = fl.data.states;
this.cities = fl.data.cities;
this.product_list = fl.data.products;

},


searchIt(){
this.$axios.get('search_order/'+this.search)
.then(res => {
this.orders = res.data.orders;            
this.data = res.data;
});
},    
get_stock_info (e) {
switch(e) {
case 'Stock does not exist':
return {class:'stock_does_not_exist',text:'Stock does not exist'}
case 0:
return {class:'out_of_stock',text:'out of stock'}
default:
return {class:'in_stock',text:'in stock'}

} 
},    
myBtnClass(name) {
switch(name) {

case 'pending':
return 'warning darken-3'
case 'processing':
return 'primary'
case 'loaded':
return 'teal'
case 'on the way':
return 'blue lighten-1'
case 'delivered':
return 'green lighten-1'
default:
return 'error'

}

}, 


get_product (i,order_item_id) {


var payload = {
order_id : this.editedItem.id,
order_item_id : order_item_id,
product_id : this.order_item.id[i],
item_quantity : this.order_item_quantity[i]
};
//console.log(payload)

this.$axios.post('change_order_item',payload)
.then(res =>{
this.editedItem.order_gross = this.get_decimal_value(res.data.updated_record.order_gross)
this.editedItem.order_tax = this.get_decimal_value(res.data.updated_record.order_tax)
this.editedItem.order_total = this.orders[this.editedIndex].order_total = this.get_decimal_value(res.data.updated_record.order_total)
Object.assign(this.editedItem.products[i],res.data.updated_record.products)

this.snackbar = res.data.response_status;
this.response.msg = res.data.message;
this.order_item.id[i] = ''
this.order_item_quantity[i] = ''

}).catch(error => console.log(error));

},

sum_of_product_price (a,b) {
return this.get_decimal_value(a * b)    
},
get_decimal_value (value) {
return (Math.round(value * 100) / 100).toFixed(2);
},


async paginate (e) {

const orderBy =  this.options.sortBy.length == 0 ? 'id' : this.options.sortBy[0];
const sortBy =  this.options.sortDesc.length > 0 || this.options.sortDesc[0]    
? 'desc' : 'asc';

var filter = '';
if(this.product_id == '' && 
this.timestamp == ''  &&
this.status_id == ''  &&
this.customer_id == ''&& 
this.driver_id == ''  &&
this.state_id == ''   &&
this.city_id == '')
{

let  data = await this.$axios.get('export_orders',{params:{
'sort_by':sortBy,
'order_by':orderBy}});
this.orders = data.data;
this.data = data.data;

}
else{
this.$axios.get('export_orders',{params:{
'sort_by':sortBy,
'order_by':orderBy,
'timestamp' : this.timestamp ,
'driver' : this.driver_id,
'status' : this.status_id,
'customer' : this.customer_id,
'state' : this.state_id,
'city' : this.city_id,
'product' : this.product_id
}})
.then(res => {
this.orders = res.data;
this.data = res.data;
});
}


},
async initialize () {

const drivers = await this.$axios.get('driver');
this.drivers = drivers.data;

const order_items = await this.$axios.get('product');
this.order_items = order_items.data;


},

editItem (item) {

this.editedIndex = this.orders.indexOf(item)
this.editedItem = Object.assign({}, item)
this.dialog = true
this.action = 'Edit Item';
},

viewItem (item) {
this.editedIndex = this.orders.indexOf(item)
this.editedItem = Object.assign({}, item)
this.dialog = true
this.action = 'View Item';
},


deleteItem (item) {
confirm('Are you sure you want to delete this item?') &&
this.$axios.delete('order/'+item.id)
.then((res) => {
if(res.data.response_status){
const index = this.orders.indexOf(item)
this.orders.splice(index, 1)
this.snackbar = res.data.response_status;
this.response.msg = res.data.message;

}
});
},

close () {
this.dialog = false
setTimeout(() => {
this.editedItem = Object.assign({}, this.defaultItem)
this.editedIndex = -1
}, 300)
},

save () {

// const stock_check = this.editedItem.products.filter((v => v.stock == 0 || v.stock == 'Stock does not exist'));
//       stock_check ? true : false
//if(this.$refs.form.validate() && stock_check == false){
if(this.$refs.form.validate()){

const payload = {
driver_id : this.editedItem.driver_id,
order_id : this.editedItem.id,
// scheduling:this.date2,
// payment_due_date:this.date,
status_id : 2
};

this.$axios.post('status_change',payload)
.then(res => {

this.orders[this.editedIndex].status = res.data.updated_record.status

this.snackbar = res.data.response_status;
this.response.msg = res.data.message;
this.close()

})
.catch(error => console.log(error));
}
else{
this.snackbar = true;
this.response.msg = 'Kindly adjust your stock';
}
},
},
}
</script>
<style>
/* i.v-icon.notranslate.mdi.mdi-radiobox-blank.theme--light{
color:white;
} */
/* label.v-label.theme--light {
color:white;
} */
.indicator{
margin-left: -24%; padding:0;
background: red !important;
width: 3px;
}
.stock_does_not_exist{
font-size: 11px;
display: block;
background: #fbdede;
color: #cc0000;
border-radius: 20px;
/* width: 120px; */
text-align: center;
}
.out_of_stock{
font-size: 11px;
display: block;
background: #fbdede;
color: #cc0000;
border-radius: 20px;
/* width: 120px; */
text-align: center;
}
.in_stock{
font-size: 11px;
display: block;
background: #ddf9be;
color: #207910;
border-radius: 20px;
text-align: center;
}
</style>