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
Orders By Driver Report 
<v-spacer></v-spacer>
   <VueJsonToCsv
    :json-data="orders"
    :labels="{ 
      id:{ title: 'Order ID' },
      driver_name:{ title: 'Driver Name' },
      status:{ title: 'Order Status' },
      order_total:{ title: 'Order Amount' },
      order_confirmed_date:{ title: 'Assigned Date' },
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
item-text="name" 
label="Drivers"
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

<v-btn @click="filter_records"  class="black white--text">
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


<template  v-slot:item.status="{ item }">
<v-chip small class="white--text" :class="myBtnClass(item.status)">
{{item.status}}
</v-chip>
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
text: 'Driver Name',
align: 'left',
value: 'driver_name',
sortable:false,
},

{
text: 'Status',
align: 'left',
value: 'status',
sortable:false,
},
{
text: 'Order Amount',
align: 'left',
value: 'order_total', 
sortable:false,
},

{
text: 'Assigned Date',
align: 'left',
value: 'order_confirmed_date',
sortable:false,
},



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
 this.myBtnClass()
},
methods: {
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
  reset () {
  this.date_from = '';
  this.date_to = '';
  this.id = '';
  this.filter_records()
},

  async get_data () {
        var all = [];
        const fl = await this.$axios.get('filter_listing'); 

        all = [{id:'',name:'All'}];
        fl.data.drivers.unshift(all[0]);
        this.list = fl.data.drivers;   
  },

  filter_records(){

const orderBy =  this.options.sortBy.length == 0 ? 'id' : this.options.sortBy[0];
const sortBy =  this.options.sortDesc.length > 0 || this.options.sortDesc[0]    
? 'asc' : 'desc';


var payload = {params:{
'sort_by':sortBy,
'order_by':orderBy,
'driver' : this.id,
'from' : this.date_from,
'to' : this.date_to,
}};

this.$axios.get('orders_by_drivers',payload)
.then(res => {


        var drivers = res.data.map(v => ({
            id:v.id,
            driver_name:v.driver[0].name,
            status:v.status,
            order_total:v.order_total,
            order_confirmed_date:v.order_confirmed_date
        }))


        this.orders = drivers;

});

},




async paginate () {
  this.filter_records()
},
},
}
</script>
