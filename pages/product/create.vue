<template>
    <v-app>
        <div class="text-center ma-2">
            <v-snackbar
                v-model="snackbar"
                top="top"
                color="success"
                elevation="24"
            >
                {{ msg }}
            </v-snackbar>
        </div>

        <v-card>
            <v-form ref="form" lazy-validation>
                <v-card-text>
                    <v-row>
                        <v-col>
                            <div class="display-1 pa-2">Add Product</div>
                        </v-col>
                        <v-col>
                            <div class="display-1 pa-2 text-right">
                                <v-btn class="primary" to="/product">
                                    <v-icon>mdi-arrow-left</v-icon>&nbsp;Back
                                </v-btn>
                            </div>
                        </v-col>
                    </v-row>

                    <v-container>
                        <v-row>
                            <v-col cols="4">
                                <v-text-field
                                    :rules="Rules"
                                    v-model="product_title"
                                    label="Product Name*"
                                ></v-text-field>
                            </v-col>
                            <v-col cols="4">
                                <v-text-field
                                    type="number"
                                    :rules="Rules"
                                    v-model="product_price"
                                    label="Product Price*"
                                ></v-text-field>
                            </v-col>
                            <!-- <v-col cols="6">
                                <v-select
                                    :rules="Rules"
                                    v-model="product_type_id"
                                    :items="product_types"
                                    item-value="id"
                                    item-text="type"
                                    label="Product Type"
                                ></v-select>
                            </v-col> -->
                            <v-col cols="4">
                                <v-select
                                    :rules="Rules"
                                    v-model="product_category_id"
                                    :items="categories"
                                    item-value="id"
                                    item-text="category"
                                    label="Product Category*"
                                ></v-select>
                            </v-col>
                        </v-row>
                        <v-row>
                            <v-col cols="12">
                                <v-textarea
                                    v-model="product_description"
                                    label="Product Description"
                                ></v-textarea>
                            </v-col>
                        </v-row>
                        <v-row>
                            <v-col cols="12">
                                <v-checkbox
                                    color="primary"
                                    v-model="IsActive"
                                    label="Is Active"
                                ></v-checkbox>
                            </v-col>
                            <v-col cols="12" class="mt-4">
                                <v-btn
                                    small
                                    :rules="Rules"
                                    class="primary accent--text"
                                    @click="onPick_soi_attachment"
                                    >{{
                                        !product_image.name
                                            ? "Upload Product Image"
                                            : product_image.name
                                    }}
                                    <v-icon right dark
                                        >mdi-cloud-upload</v-icon
                                    ></v-btn
                                ><span class="pa-3 title primary--text">*</span>
                                <div class="display-9 pt-3">
                                    Supported Image Types : JPEG, JPG, PNG, GIF
                                </div>
                                <div class="display-9">
                                    Preferred Dimensions : 600x475
                                </div>

                                <input
                                    required
                                    type="file"
                                    @change="check_soi_attachment"
                                    style="display: none"
                                    accept="image/*"
                                    ref="soi_attachmentInput"
                                />
                            </v-col>
                            <v-col v-if="errors && errors.length > 0" cols="12">
                                <ul>
                                    <li
                                        class="error--text"
                                        v-for="(err, i) in errors"
                                        :key="i"
                                    >
                                        {{ err }}
                                    </li>
                                </ul>
                            </v-col>
                            <v-col>
                                <v-btn
                                    small
                                    color="primary"
                                    class="mr-4 accent--text"
                                    @click="save"
                                >
                                    Submit
                                </v-btn>
                            </v-col>
                        </v-row>
                    </v-container>
                </v-card-text>
            </v-form>

            <template v-slot:item.action="{ item }">
                <v-icon small class="mr-2" @click="editItem(item)">
                    mdi-pencil
                </v-icon>
                <v-icon small @click="deleteItem(item)"> mdi-delete </v-icon>
            </template>
        </v-card>
    </v-app>
</template>
</v-card>
</v-app>
</template>


<script>
export default {
    data: (vm) => ({
        date: new Date().toISOString().substr(0, 10),
        expiry_date: vm.formatDate(new Date().toISOString().substr(0, 10)),
        menu1: false,
        categories: [],
        product_types: [],
        product_title: "",
        product_price: "",
        product_image: "",
        product_category_id: "",
        product_type_id: "",
        product_description: "",
        IsActive: "",
        msg: "",
        snackbar: false,
        Rules: [(v) => !!v || "This field is required"],
        errors: [],
    }),
    computed: {
        computedexpiry_date() {
            return this.formatDate(this.date);
        },
    },
    watch: {
        date(val) {
            this.expiry_date = this.formatDate(this.date);
        },
    },
    created() {
        this.$axios
            .get("category")
            .then((res) => {
                this.categories = res.data.data;
            })
            .catch((err) => this.err);

        this.$axios
            .get("product_type")
            .then((res) => {
                this.product_types = res.data.data;
            })
            .catch((err) => this.err);
    },
    methods: {
        formatDate(date) {
            if (!date) return null;

            const [year, month, day] = date.split("-");
            return `${month}/${day}/${year}`;
        },
        parseDate(date) {
            if (!date) return null;

            const [month, day, year] = date.split("/");
            return `${year}-${month.padStart(2, "0")}-${day.padStart(2, "0")}`;
        },

        onPick_soi_attachment() {
            this.$refs.soi_attachmentInput.click();
        },

        check_soi_attachment(e) {
            this.product_image = e.target.files[0] || "";
        },

        save() {
            let product = new FormData();
            product.append("product_title", this.product_title.toLowerCase());
            product.append("product_price", this.product_price);
            product.append("product_image", this.product_image);
            product.append("category_id", this.product_category_id);
            product.append("product_type_id", this.product_type_id);
            product.append("product_description", this.product_description);
            product.append("IsActive", this.IsActive == true ? 1 : 0);
            this.$axios.post("product", product).then((res) => {
                if (res.data.status) {
                    this.snackbar = res.data.status;
                    this.msg = "Prodcut has been added";
                    setTimeout(() => this.$router.push("/product"), 2000);
                } else {
                    this.errors = res.data.errors;
                }
            });
        },
    },
};
</script>