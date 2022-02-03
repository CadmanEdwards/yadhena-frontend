<template>
<v-app>
<v-data-table
:headers="headers"
:items="orders"
class="elevation-1"
:options.sync="options"
@pagination="paginate"
:items-per-page=10
:footer-props="{
itemsPerPageOptions:[10]
}"
>


<template v-slot:top>
<v-toolbar class="primary accent--text title" flat>
Orders By Customer Report 
<v-spacer></v-spacer>
   <VueJsonToCsv
    :json-data="orders"
    :labels="{ 
      id:{ title: 'Order ID' },
      company_name:{ title: 'Company Name' },
      contact_person_name:{ title: 'Contact Person' },
      order_total:{ title: 'Order Amount' },
      created_at:{ title: 'Ordered DateTime' },
      }"    

    >
    <v-btn class="primary mx-2 accent--text no_print">
    <v-icon>mdi-file-export</v-icon><b>&nbsp;Export CSV </b>
    </v-btn>
    </VueJsonToCsv>
</v-toolbar>



<v-row class="px-5"> 
<v-col>
<v-select
v-model="id" 
:items="list"
item-value="id"
item-text="company_name" 
label="Customers"
></v-select>
</v-col>
</v-row>

<v-row class="px-4 mt-2">
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
            label="From"
            v-model="date_from"
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
            label="To"
            dense
            v-model="date_to"
            readonly
            v-on="on"
          ></v-text-field>
        </template>
        
        <v-date-picker color="primary accent--text" v-model="date_to" @input="menu_to = false"></v-date-picker>
      </v-menu>
    </v-col>
</v-row>
<v-row class="px-4" style="">
  
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
</template>
<template v-slot:item.order_total="{ item }">
{{item.order_total | get_decimal_value}}
</template>


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

id:'',    

options:{
sortBy:['id','order_total','created_at'],
sortDesc:[true]
},

headers: [
{
text: 'Order #',
align: 'left',
value: 'id',
sortable:false,
},

{
text: 'Company Name',
align: 'left',
value: 'company_name',
sortable:false,
},
{
text: 'Contact Person',
align: 'left',
value: 'contact_person_name',
sortable:false,
},
{
text: 'Order Amount',
align: 'left',
value: 'order_total',
sortable:false,
},
{
text: 'Ordered DateTime',
align: 'left',
value: 'created_at',
sortable:false,
}
],

orders:[],
list:[],
}),

computed: {

},
filters: {
  get_decimal_value: function (value) {
    if (!value) return ''
    return (Math.round(value * 100) / 100).toFixed(2);
  }
},
created () {
 this.get_data()
},
methods: {

  reset () {
  this.date_from = '';
  this.date_to = '';
  this.id = '';
  this.filter_records()
},

  async get_data () {
        var all = [];
        const fl = await this.$axios.get('filter_listing'); 

        all = [{id:'',company_name:'All'}];
        fl.data.customers.unshift(all[0]);
        this.list = fl.data.customers;   
  },

  filter_records(){

const orderBy =  this.options.sortBy.length == 0 ? 'id' : this.options.sortBy[0];
const sortBy =  this.options.sortDesc.length > 0 || this.options.sortDesc[0]    
? 'asc' : 'desc';


var payload = {params:{
'sort_by':sortBy,
'order_by':orderBy,
'customer' : this.id,
'from' : this.date_from,
'to' : this.date_to,
}};

this.$axios.get('orders_by_customers',payload)
.then(res => {
this.orders = res.data;
console.log(this.orders);
});

},




async paginate () {
  this.filter_records()
},
},
}
</script>
