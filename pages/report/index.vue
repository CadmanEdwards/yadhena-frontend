<template>
<div>
<v-row class="mt-10">
<v-col :cols="col" v-for="card in cards" :key="card.id">
<v-card :to="'report/'+card.link" class="no_print">

<v-list-item>
<v-list-item-content>
<div class="overline mb-4">Report</div>
<v-list-item-title color="pink darken-2" class=" title mb-1">{{card.name}}</v-list-item-title>
<v-list-item-subtitle>
</v-list-item-subtitle>
</v-list-item-content>
<v-list-item-avatar
tile class="primary" size="60">
<span>
  <v-icon class="white--text">{{card.icon}}</v-icon>
</span>
</v-list-item-avatar>
</v-list-item>
</v-card>


</v-col> 


</v-row>   

</div>
</template>
<script>
export default {
  data:() => ({
      col:4,
      cards : [
      { id:1, name:'Sales',icon:'mdi-trending-up',link:'sales'},
      { id:2, name:'Order',icon:'mdi-package-variant',link:'orders'},
      { id:3, name:'Order By Product',icon:'mdi-nutrition',link:'products'},
      { id:4, name:'Order By Customer',icon:'mdi-account',link:'customers'},
      { id:5, name:'Order By Driver',icon:'mdi-truck-delivery',link:'drivers'},
      { id:6, name:'Order By Status',icon:'mdi-truck-delivery',link:'statusses'},
      { id:7, name:'Order By State',icon:'mdi-city',link:'states'},
      { id:8, name:'Order By City',icon:'mdi-city',link:'cities'},
      { id:9, name:'Repair',icon:'mdi-format-page-break',link:'repairs'},
      { id:10, name:'Stock Adjustment',icon:'mdi-package-variant',link:'stock_adjustments'}
      ]
  })
}
</script>