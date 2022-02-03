<template>
<v-app>
<div class="text-center ma-2">

<v-snackbar
v-model="snackbar"
:top="'top'"
>
{{response.msg}}
<v-btn     
small 
text
@click="snackbar = false"
>
Close
</v-btn>
</v-snackbar>
</div>
<v-data-table
:headers="headers"
:items="products"
:search="search"
class="elevation-1"
>
<template  v-slot:item.product_image="{ item }">
<br>
<img width="50" :src="item.product_image"/>
</template>
<template v-slot:top>
<v-toolbar flat>
<v-toolbar-title>Products</v-toolbar-title>
<v-divider
class="mx-4"
inset
vertical
></v-divider>

<v-text-field
v-model="search"
label="Search"
single-line
hide-details
></v-text-field>
<v-divider
class="mx-4"
inset
vertical
></v-divider>
<v-dialog v-model="dialog" max-width="1000px">

	<template v-slot:activator="{  }">
	<v-btn small class="mb-2 primary accent--text" to="/product/create">Add Product</v-btn>
	</template>
<v-card>
<v-card-title>

<v-icon small :color="editedItem.IsActive ? 'success' : 'error' ">mdi-checkbox-blank-circle</v-icon>
&nbsp;{{editedItem.IsActive ? 'Active' : 'Inactive'}}
<v-spacer></v-spacer>
<img  height="75" width="75" :src="editedItem.product_image" alt="Orange Room Digital" />
</v-card-title>

<v-card-text>
<v-container>
<v-form ref="form" lazy-validation>
<v-row>
<v-col>
<v-text-field :rules="Rules" :readonly="isReadOnly" v-model="editedItem.product_title" label="Product Name"></v-text-field>
</v-col>
<v-col>
 <v-select    
	v-if="!isReadOnly"
	v-model="editedItem.category_id" 
	:items="categories"
	item-text="category" 
	item-value="id"
	label="Product Category"
	persistent-hint
 ></v-select>
<v-text-field :readonly="isReadOnly"  v-else v-model="editedItem.category" label="Product Category"></v-text-field>
</v-col>
</v-row>
<v-row>
</v-row>

<v-row>


<v-col>
<v-text-field :rules="Rules" :readonly="isReadOnly" v-model="editedItem.product_price" label="Product Price"></v-text-field>
</v-col>

</v-row>

<v-row>
<v-col>
<v-text-field :rules="Rules" :readonly="isReadOnly" v-model="editedItem.product_description" label="product Description"></v-text-field>
</v-col>
</v-row>
<v-row v-if="!isReadOnly">
<v-col>
<v-checkbox
color="primary"
:rules="Rules"
v-model="editedItem.IsActive"
label="Is Active"
></v-checkbox>
</v-col>
</v-row>
<v-row>
  
    <v-col v-if="!isReadOnly">
    <v-btn small raised class="primary accent--text" @click="upload_btn">Upload Image</v-btn>  
    <input required type="file" @change="upload_trigger" style="display:none;" accept="image/*" ref="attachment">
</v-col>
<v-spacer></v-spacer>
<v-btn small class="primary accent--text mt-4" text @click="close">Cancel</v-btn>
&nbsp;
<v-btn small v-if="!isReadOnly" class="primary accent--text mt-4" text @click="save">Save</v-btn>


</v-row>
</v-form>
</v-container>
</v-card-text>

</v-card>

</v-dialog>
</v-toolbar>
</template>
<template v-slot:item.action="{ item }">
<v-icon
small
class="mr-2"
@click="viewItem(item)"
>
mdi-eye
</v-icon>

<v-icon
small
class="mr-2"
@click="editItem(item)"
>
mdi-pencil
</v-icon>
</template>
<template v-slot:no-data>
</template>
</v-data-table>
</v-app>
</template>
<script>
export default {

data: () => ({
action:'',
search:'',
snackbar:false,
dialog: false,
headers: [
{
text: 'Image',
align: 'left',
sortable: false,
value: 'product_image',
},
{
text: 'Product Title',
align: 'left',
sortable: false,
value: 'product_title',
},
{
text: 'Price',
align: 'left',
sortable: false,
value: 'product_price',
},
{
text: 'Category',
align: 'left',
sortable: false,
value: 'category.category',
},
{ text: 'Actions', value: 'action', sortable: false },
],
editedIndex: -1,
editedItem: {
group_code:'',
group_description:'',
pack_code:'',
pack_description:'',
legacy_code_sku: '',
product_title: '',
product_price: '',

upload_image: {
  name:''
},

// product_category_id: '',
unit_of_measurement:'',
product_description:'',
unit_in_case:'',
weight:'',
expiry_date:'',
IsActive:'',

},
defaultItem: {
group_code:'',
group_description:'',
pack_code:'',
pack_description:'',  
legacy_code_sku: '',
product_title: '',
product_price: '',

upload_image: {
  name:''
},

// product_category_id: '',
unit_of_measurement:'',
product_description:'',
unit_in_case:'',
weight:'',
expiry_date:'',
IsActive:'',
},
response : {
msg:''
},
products:[],
categories:[],
Rules : [
  v => !!v || 'This field is required',
],
fileRules: [
value => !value || value.size < 2000000 || 'Image size should be less than 2 MB!',
],
}),

computed: {
isReadOnly(){
    return this.action == 'View Item'
},
formTitle () {
return (this.action) ? this.action : this.editedIndex === -1 ? 'New Product' : 'Edit Product'
},  

},

watch: {
dialog (val) {
val || this.close()
},
},

async created () {
const products = await this.$axios.get('product');
this.products = products.data;
},

methods: {
upload_btn () { 
  this.$refs.attachment.click() 
},

upload_trigger(e) { 

  const file = e.target.files[0];
  this.editedItem.upload_image = file || ''; 
},

async editItem (item) {
this.editedIndex = this.products.indexOf(item)
this.editedItem = Object.assign({}, item)
const categories = await this.$axios.get('category');
this.categories = categories.data.data || [];


this.dialog = true
this.action = 'Edit Item';
},

viewItem (item) {

this.editedIndex = this.products.indexOf(item)
this.editedItem = Object.assign({}, item)
this.editedItem.category = this.editedItem.category.category
this.dialog = true
this.action = 'View Item';
},

close () {
this.dialog = false


setTimeout(() => {
this.editedItem = Object.assign({}, this.defaultItem)
this.editedIndex = -1
}, 300)
},

save () {


  let product = new FormData();
  
  product.append('legacy_code_sku',this.editedItem.legacy_code_sku);
  product.append('group_code',this.editedItem.group_code);
  product.append('group_description',this.editedItem.group_description);
  product.append('pack_code',this.editedItem.pack_code);
  product.append('pack_description',this.editedItem.pack_description);
  product.append('product_title',this.editedItem.product_title);
  product.append('product_price',this.editedItem.product_price);
  product.append('product_image',this.editedItem.upload_image || this.editedItem.product_image);
  // product.append('product_category_id',this.editedItem.product_category_id);
  product.append('unit_of_measurement',this.editedItem.unit_of_measurement);
  product.append('product_description',this.editedItem.product_description);
  product.append('unit_in_case',this.editedItem.unit_in_case);
  product.append('weight',this.editedItem.weight);
  product.append('expiry_date',this.editedItem.expiry_date);
  product.append('IsActive',this.editedItem.IsActive == true ? 1 : 0);



if(this.$refs.form.validate()){
this.$axios.post('product/' + this.editedItem.id, product)
.then(res => {

if(res.data.response_status){ 
    const index = this.products.findIndex(item => item.id == this.editedItem.id)

    Object.assign(this.products[index],res.data.updated_record);
    this.snackbar = res.data.response_status;
    this.response.msg = res.data.message;
    this.close()

}
})
.catch(error => console.log(error));
}
}
},

}
</script>