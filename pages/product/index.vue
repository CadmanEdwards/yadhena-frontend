<template>
    <v-app>
        <div class="text-center ma-2">
            <v-snackbar
                v-model="snackbar"
                top="top"
                color="success"
                elevation="24"
            >
                {{ response.msg }}
            </v-snackbar>
        </div>

        <v-data-table
            v-model="ids"
            show-select
            item-key="id"
            :headers="headers"
            :items="products"
            :search="search"
            :server-items-length="total"
            :loading="loading"
            @pagination="paginate"
            :footer-props="{
                itemsPerPageOptions: [5, 10, 15],
            }"
            class="elevation-1"
        >
            <template v-slot:item.product_image="{ item }">
                <br />
                <img
                    width="50"
                    :src="item.product_image"
                    @click="showImage(item.product_image)"
                />
            </template>
            <template v-slot:top>
                <v-toolbar flat>
                    <v-toolbar-title>Products</v-toolbar-title>
                    <v-divider class="mx-4" inset vertical></v-divider>

                    <v-text-field
                        v-model="search"
                        label="Search"
                        single-line
                        hide-details
                    ></v-text-field>
                    <v-divider class="mx-4" inset vertical></v-divider>
                    <v-btn
						v-if="can('product_delete')"
                        small
                        color="error"
                        class="mr-2 mb-2"
                        @click="delteteSelectedRecords"
                        >Delete Selected Records</v-btn
                    >
                    <v-dialog v-model="dialog" max-width="1000px">
                        <template v-slot:activator="{}">
                            <v-btn
								v-if="can('product_create')"
                                small
                                class="mb-2 success accent--text"
                                to="/product/create"
                                >Product +</v-btn
                            >
                        </template>
                        <v-card>
                            <v-card-title>
                                <v-icon
                                    small
                                    :color="
                                        editedItem.IsActive
                                            ? 'success'
                                            : 'error'
                                    "
                                >
                                    mdi-checkbox-blank-circle
                                </v-icon>
                                &nbsp;{{
                                    editedItem.IsActive ? "Active" : "Inactive"
                                }}
                                <v-spacer></v-spacer>
                            </v-card-title>

                            <v-card-text>
                                <v-container>
                                    <v-form ref="form" lazy-validation>
                                        <v-row>
                                            <v-col cols="6">
                                                <v-text-field
                                                    :rules="Rules"
                                                    :readonly="isReadOnly"
                                                    v-model="
                                                        editedItem.product_title
                                                    "
                                                    label="Product Name"
                                                ></v-text-field>
                                            </v-col>
                                            <v-col cols="6">
                                                <v-text-field
                                                    :rules="Rules"
                                                    :readonly="isReadOnly"
                                                    v-model="
                                                        editedItem.product_price
                                                    "
                                                    label="Product Price"
                                                ></v-text-field>
                                            </v-col>

                                            <v-col cols="6">
                                                <v-select
                                                    v-if="!isReadOnly"
                                                    v-model="
                                                        editedItem.category_id
                                                    "
                                                    :items="categories"
                                                    item-text="category"
                                                    item-value="id"
                                                    label="Product Category"
                                                    persistent-hint
                                                ></v-select>
                                                <v-text-field
                                                    :readonly="isReadOnly"
                                                    v-else
                                                    v-model="
                                                        editedItem.category
                                                    "
                                                    label="Product Category"
                                                ></v-text-field>
                                            </v-col>
                                        </v-row>

                                        <v-row>
                                            <v-col>
                                                <v-text-field
                                                    :rules="Rules"
                                                    :readonly="isReadOnly"
                                                    v-model="
                                                        editedItem.product_description
                                                    "
                                                    label="product Description"
                                                ></v-text-field>
                                            </v-col>
                                        </v-row>
                                        <v-row v-if="!isReadOnly">
                                            <v-col>
                                                <v-checkbox
                                                    color="primary"
                                                    :rules="Rules"
                                                    v-model="
                                                        editedItem.IsActive
                                                    "
                                                    label="Is Active"
                                                ></v-checkbox>
                                            </v-col>
                                        </v-row>
                                        <v-row>
                                            <v-col v-if="!isReadOnly">
                                                <v-btn
                                                    small
                                                    raised
                                                    class="primary accent--text"
                                                    @click="upload_btn"
                                                    >Upload Image
                                                    <v-icon right dark
                                                        >mdi-cloud-upload</v-icon
                                                    >
                                                </v-btn>
                                                <input
                                                    required
                                                    type="file"
                                                    @change="upload_trigger"
                                                    style="display: none"
                                                    accept="image/*"
                                                    ref="attachment"
                                                />
                                            </v-col>
                                            <v-spacer></v-spacer>
                                            <v-btn
                                                small
                                                class="
                                                    primary
                                                    accent--text
                                                    mt-4
                                                "
                                                text
                                                @click="close"
                                                >Cancel</v-btn
                                            >
                                            &nbsp;
                                            <v-btn
                                                small
                                                v-if="!isReadOnly"
                                                class="
                                                    primary
                                                    accent--text
                                                    mt-4
                                                "
                                                text
                                                @click="save"
                                            >
                                                Save
                                            </v-btn>
                                        </v-row>
                                    </v-form>
                                </v-container>
                            </v-card-text>
                        </v-card>
                    </v-dialog>
                </v-toolbar>
            </template>

            <template v-slot:item.action="{ item }">
                <v-btn
					v-if="can('product_read')"
                    x-small
                    class="primary mr-2"
                    @click="showImage(item.product_image)"
                >
                    View Image
                </v-btn>
                <v-icon small color="info" class="mr-2" @click="viewItem(item)">
                    mdi-eye
                </v-icon>
                <v-icon
					v-if="can('product_edit')"
                    small
                    color="secondary"
                    class="mr-2"
                    @click="editItem(item)"
                >
                    mdi-pencil
                </v-icon>
                <v-icon
					v-if="can('product_delete')"
                    small
                    color="error"
                    class="mr-2"
                    @click="deleteItem(item)"
                >
                    mdi-delete
                </v-icon>
            </template>
        </v-data-table>
        <template>
            <div class="text-center">
                <v-dialog v-model="imageModal" width="500">
                    <v-card>
                        <v-card-text class="pa-3">
                            <v-img :src="setImage" />
                        </v-card-text>

                        <v-divider></v-divider>

                        <v-card-actions>
                            <v-spacer></v-spacer>
                            <v-btn
                                small
                                class="primary accent--text mt-4"
                                text
                                @click="imageModal = false"
                                >Cancel
                            </v-btn>
                        </v-card-actions>
                    </v-card>
                </v-dialog>
            </div>
        </template>
    </v-app>
</template>
<script>
export default {
    data: () => ({
        action: "",
        search: "",
        loading: false,
        snackbar: false,
        dialog: false,
        imageModal: false,
        setImage: "",
        total: 0,
        ids: [],
        headers: [
            {
                text: "Image",
                align: "left",
                sortable: false,
                value: "product_image",
            },
            {
                text: "Product Title",
                align: "left",
                sortable: false,
                value: "product_title",
            },
            {
                text: "Price",
                align: "left",
                sortable: false,
                value: "product_price",
            },
            {
                text: "Category",
                align: "left",
                sortable: false,
                value: "category.category",
            },
            { text: "Actions", value: "action", sortable: false },
        ],
        editedIndex: -1,
        editedItem: {
            product_title: "",
            product_price: "",
            upload_image: { name: "" },
            product_description: "",
            IsActive: "",
            category_id: "",
        },
        defaultItem: {
            product_title: "",
            product_price: "",
            upload_image: { name: "" },
            product_description: "",
            IsActive: "",
            category_id: "",
        },
        response: { msg: "" },
        products: [],
        categories: [],
        Rules: [(v) => !!v || "This field is required"],
        fileRules: [
            (value) =>
                !value ||
                value.size < 2000000 ||
                "Image size should be less than 2 MB!",
        ],
    }),

    computed: {
        isReadOnly() {
            return this.action == "View Item";
        },

        formTitle() {
            return this.action
                ? this.action
                : this.editedIndex === -1
                ? "New Product"
                : "Edit Product";
        },
    },

    watch: {
        dialog(val) {
            val || this.close();
            this.errors = [];
            this.search = "";
        },
    },

    async created() {
        this.loading = true;
        const categories = await this.$axios.get("category");
        this.categories = categories.data.data;
    },

    methods: {
        can(permission) {
            let user = this.$auth.user;
            return (
                (user &&
                    user.permissions.some((e) => e.permission == permission)) ||
                user.master
            );
        },
        async paginate(e) {
            this.$axios
                .get("product?page=" + e.page, {
                    params: { per_page: e.itemsPerPage },
                })
                .then((res) => {
					this.products = this.can('product_read') ? res.data.data : [];
                    this.total = this.can('product_read') ? res.data.total : 0;
                    this.loading = false;
                });
        },

        showImage(img) {
            this.imageModal = true;
            this.setImage = img;
        },
        upload_btn() {
            this.$refs.attachment.click();
        },

        upload_trigger(e) {
            const file = e.target.files[0];
            this.editedItem.upload_image = file || "";
        },

        async editItem(item) {
            this.editedIndex = this.products.indexOf(item);
            this.editedItem = Object.assign({}, item);
            this.editedItem.category_id = parseInt(item.category_id);

            this.dialog = true;
            this.action = "Edit Item";
        },

        delteteSelectedRecords() {
            let just_ids = this.ids.map((e) => e.id);
            confirm(
                "Are you sure you wish to delete selected records , to mitigate any inconvenience in future."
            ) &&
                this.$axios
                    .post("product-dsr", {
                        ids: just_ids,
                    })
                    .then((res) => {
                        if (!res.data.status) {
                            this.errors = res.data.errors;
                        } else {
                            this.$axios.get("product").then((res) => {
                                this.products = res.data.data;
                                this.total = res.data.total;
                                this.snackbar = true;
                                this.ids = [];
                                this.response.msg =
                                    "Selected records has been deleted";
                            });
                        }
                    })
                    .catch((err) => console.log(err));
        },

        deleteItem(item) {
            confirm("Are you sure you want to delete this item?") &&
                this.$axios.delete("product/" + item.id).then((res) => {
                    const index = this.products.indexOf(item);
                    this.products.splice(index, 1);
                    this.snackbar = true;
                    this.response.msg = "Product has been deleted";
                });
        },

        viewItem(item) {
            this.editedIndex = this.products.indexOf(item);
            this.editedItem = Object.assign({}, item);
            this.editedItem.category = this.editedItem.category.category;
            this.dialog = true;
            this.action = "View Item";
        },

        close() {
            this.dialog = false;

            setTimeout(() => {
                this.editedItem = Object.assign({}, this.defaultItem);
                this.editedIndex = -1;
            }, 300);
        },

        save() {
            let product = new FormData();

            product.append(
                "product_title",
                this.editedItem.product_title.toLowerCase()
            );
            product.append("product_price", this.editedItem.product_price);
            product.append(
                "product_image",
                this.editedItem.upload_image || this.editedItem.product_image
            );
            product.append("category_id", this.editedItem.category_id);
            product.append(
                "product_description",
                this.editedItem.product_description
            );
            product.append(
                "IsActive",
                this.editedItem.IsActive == true ? 1 : 0
            );

            if (this.$refs.form.validate()) {
                this.$axios
                    .post("product/" + this.editedItem.id, product)
                    .then((res) => {
                        if (!res.data.status) {
                            this.errors = res.data.errors;
                        } else {
                            const idx = this.products.findIndex(
                                (item) => item.id == this.editedItem.id
                            );
                            Object.assign(this.products[idx], res.data.record);
                            this.snackbar = res.data.status;
                            this.response.msg = res.data.message;
                            this.close();
                        }
                    })
                    .catch((error) => console.log(error));
            }
        },
    },
};
</script>