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
Stock Adjustment Report 
<v-spacer></v-spacer>
   <VueJsonToCsv
    :json-data="orders"
    :labels="{ 
      id:{ title: '#' },
      company_name:{ title: 'Company Name' },
      keyword:{ title: 'Status' },  
      schedule:{ title: 'Scheduled Date' },      
      created_at:{ title: 'DateTime' },
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
label="Customer"
></v-select>
</v-col>
</v-row>
<v-row class="px-5">
  <v-col>
<v-radio-group v-model="keyword" row>
<span>Status</span>&nbsp;&nbsp;
<v-radio color="primary accent--text" label="All" value=""></v-radio>
<v-radio color="primary accent--text" label="Open" value="1"></v-radio>
<v-radio color="primary accent--text" label="Scheduled" value="2"></v-radio>
<v-radio color="primary accent--text" label="Repaired" value="5"></v-radio>
</v-radio-group>
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

<template  v-slot:item.keyword="{ item }">
<v-chip small class="accent--text" :class="myBtnClass(item.keyword)">
{{item.keyword | capitalize}}
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
reason:'',
keyword:'',
options:{
sortBy:['id','order_total','created_at'],
sortDesc:[true]
},
 headers: [
       {
text: '#',
align: 'left',
value: 'id',
},

{
text: 'Customer Name',
align: 'left',
sortable: false,
value: 'company_name',
},
{
text: 'Status',
align: 'left',
sortable: false,
value: 'keyword',
},
{
text: 'Scheduled Date',
align: 'left',
sortable: false,
value: 'schedule',
},
{
text: 'Submitted At',
align: 'left',
sortable: false,
value: 'created_at',
},
// { text: 'Action', value: 'action', sortable: false },
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
filters: {
  capitalize (v) {
     return v.charAt(0).toUpperCase() + v.slice(1)
  }
},
methods: {
myBtnClass(name) {
switch(name) {

case 'open':
return 'warning darken-3'
case 'scheduled':
return 'primary'
case 'repaired':
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

        all = [{id:'',company_name:'All'}];
        fl.data.customers.unshift(all[0]);

        this.list = fl.data.customers;   
  },

filter_records(){

  console.log(this.keyword);

var payload = {params:{
'customer' : this.id,
'status' : this.keyword,
'scheduling' : this.scheduling,
'type' : this.type,
'to' : this.date_to,
'from' : this.date_from,
}};

this.$axios.get('maintenanceuser',payload)
.then(res => {
  this.orders = res.data;
});

},
async paginate () {
  this.filter_records()
},
},
}
</script>
