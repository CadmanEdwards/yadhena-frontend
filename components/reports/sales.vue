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
<template v-slot:item.order_total="{ item }">
{{item.order_total | get_decimal_value | get_comma_seperator}}
</template>


<template v-slot:top>
<v-toolbar class="primary title accent--text" flat>
Sales Report 
<v-spacer></v-spacer>
   <VueJsonToCsv
    :json-data="orders"
    :labels="{ 
      id:{ title: 'order id' },
      company_name:{ title: 'Company Name' },
      order_delivered_date:{ title: 'Order Delivered Date' }, 
      order_total:{ title: 'Total Sale' }, 
      created_at:{title : 'Ordered DateTime'}
      }">
    <v-btn class="primary mx-2 no_print">
    <v-icon>mdi-file-export</v-icon><b>&nbsp;Export CSV </b>
    </v-btn>
    </VueJsonToCsv>
</v-toolbar>

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
<v-icon>mdi-filter</v-icon>Filter
</v-btn>
<v-btn @click="reset"  class="accent primary--text mx-2">
<v-icon>mdi-backup-restore</v-icon>&nbsp;Reset
</v-btn>
</v-col>
</v-row>
<v-row class="px-4" style="">
  
<v-col>
<v-alert dense class="primary accent--text">
      Total Sale: <strong v-if="total_sale > 0">{{total_sale | get_decimal_value | get_comma_seperator}}</strong> 
</v-alert>

</v-col>
</v-row>
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
text: 'Customer',
align: 'left',
value: 'company_name',
sortable:false,
},

{
text: 'Delivered Date',
align: 'left',
value: 'order_delivered_date',
sortable:false,
},
{
text: 'Sales',
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

product_list:[],
orders:[],

}),

computed: {
total_sale () {
  return this.orders
              .filter(v => v.status = 'delivered')
              .reduce((total,sum) => total + parseFloat(sum.order_total) ,0);             
}
},
filters: {
  get_decimal_value: function (value) {
    if (!value) return ''
    return (Math.round(value * 100) / 100).toFixed(2);
  },
    get_comma_seperator: function (x) {
    if (!x) return ''
    return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    },
},
methods: {

reset () {
  this.date_from = '';
  this.date_to = '';
  this.filter_records()
},

filter_records(){

const orderBy =  this.options.sortBy.length == 0 ? 'id' : this.options.sortBy[0];
const sortBy =  this.options.sortDesc.length > 0 || this.options.sortDesc[0]    
? 'desc' : 'asc';


var payload = {params:{
'sort_by':sortBy,
'order_by':orderBy,
'from' : this.date_from,
'to' : this.date_to,
}};

this.$axios.get('export_orders',payload)
.then(res => {
this.orders = res.data;
this.orders = this.orders.filter(v => v.status == 'delivered');
});

},

get_decimal_value (value) {
//return (Math.round(value * 100) / 100).toFixed(2);
},

async paginate () {
  this.filter_records()
},
},
}
</script>
