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
            :items="vouchers"
            :search="search"
            :server-items-length="total"
            :loading="loading"
            @pagination="paginate"
            :footer-props="{
                itemsPerPageOptions: [5, 10, 15],
            }"
            class="elevation-1"
        >
            <template v-slot:top>
                <v-toolbar flat color="">
                    <v-toolbar-title>Vouchers</v-toolbar-title>
                    <v-divider class="mx-4" inset vertical></v-divider>

                    <v-text-field
                        v-model="search"
                        label="Search"
                        single-line
                        hide-details
                    ></v-text-field>
                    <v-divider class="mx-4" inset vertical></v-divider>
                    <v-btn
                        v-if="can('voucher_delete')"
                        small
                        color="error"
                        class="mr-2 mb-2"
                        @click="delteteSelectedRecords"
                        >Delete Selected Records</v-btn
                    >
                    <v-btn
                        v-if="can('voucher_create')"
                        small
                        color="success"
                        class="mb-2"
                        @click="dialog = true"
                        >Voucher +</v-btn
                    >
                    <v-dialog v-model="dialog" max-width="500px">
                        <v-card>
                            <v-card-title>
                                <span class="headline">{{ formTitle }}</span>
                            </v-card-title>

                            <v-card-text>
                                <v-container>
                                      <v-row>
                                        <v-col>
                                            <v-text-field
                                                v-model="editedItem.code"
                                                label="Code"
                                            ></v-text-field>
                                        </v-col>
                                        <v-col>
                                            <v-text-field
                                                v-model="
                                                    editedItem.discount_value
                                                "
                                                label="Discount Value"
                                            ></v-text-field>
                                        </v-col>
                                        <v-col cols="12">
                                            <v-menu
                                                v-model="menu2"
                                                :close-on-content-click="false"
                                                :nudge-right="40"
                                                transition="scale-transition"
                                                offset-y
                                                min-width="auto"
                                            >
                                                <template
                                                    v-slot:activator="{
                                                        on,
                                                        attrs,
                                                    }"
                                                >
                                                    <v-text-field
                                                        v-model="
                                                            editedItem.expiry_date
                                                        "
                                                        label="Expiry Date"
                                                        readonly
                                                        v-bind="attrs"
                                                        v-on="on"
                                                    ></v-text-field>
                                                </template>
                                                <v-date-picker
                                                    v-model="
                                                        editedItem.expiry_date
                                                    "
                                                    @input="menu2 = false"
                                                ></v-date-picker>
                                            </v-menu>
                                            <ul v-if="errors && errors.length > 0" class="error--text">
												<li v-for="(item, index) in errors" :key="index">
													{{ item }}
												</li>
											</ul>
                                        </v-col>
                                    </v-row>
                                </v-container>
                            </v-card-text>

                            <v-card-actions>
                                <v-spacer></v-spacer>
                                <v-btn class="error" small @click="close"
                                    >Cancel</v-btn
                                >
                                <v-btn class="primary" small @click="save"
                                    >Save</v-btn
                                >
                            </v-card-actions>
                        </v-card>
                    </v-dialog>
                </v-toolbar>
            </template>
            <template v-slot:item.action="{ item }">
                <v-icon
                    v-if="can('voucher_edit')"
                    color="secondary"
                    small
                    class="mr-2"
                    @click="editItem(item)"
                >
                    mdi-pencil
                </v-icon>
                <v-icon
                    v-if="can('voucher_delete')"
                    color="error"
                    small
                    @click="deleteItem(item)"
                >
                    mdi-delete
                </v-icon>
            </template>
            <template v-slot:no-data>
                <!-- <v-btn color="primary" @click="initialize">Reset</v-btn> -->
            </template>
        </v-data-table>
    </v-app>
</template>
<script>
export default {
  data: () => ({
	  	endpoint : "voucher",
        date: new Date(Date.now() - new Date().getTimezoneOffset() * 60000)
            .toISOString()
            .substr(0, 10),
		ids : [],
        menu2: false,
        search: "",
        snackbar: false,
        dialog: false,
		total : 0,
        headers: [
			{
                text: "Code",
                align: "left",
                sortable: false,
                value: "code",
            },
            {
                text: "Status",
                align: "left",
                sortable: false,
                value: "status",
            },
            {
                text: "Expiry Date",
                align: "left",
                sortable: false,
                value: "expiry_date",
            },
            {
                text: "Discounted Value",
                align: "left",
                sortable: false,
                value: "discount_value",
            },
            { text: "Actions", value: "action", sortable: false },
        ],
        editedIndex: -1,
        editedItem: {
            code: "",
            expiry_date: "",
            discount_value: "",
        },
        defaultItem: {
            code: "",
            expiry_date: "",
            discount_value: "",
        },
        response: {
            msg: "",
        },
        vouchers: [],
        errors: [],
    }),

    computed: {
        formTitle() {
            return this.editedIndex === -1 ? "New Voucher" : "Edit Voucher";
        },
    },

    watch: {
        dialog(val) {
            val || this.close();
            this.errors = [];
            this.search = "";
        },
    },

    created() {
        this.loading = true;
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
                .get(this.endpoint + "?page=" + e.page, {
                    params: { per_page: e.itemsPerPage },
                })
                .then((res) => {
                    this.vouchers = this.can("voucher_read") ? res.data.data : [];
                    this.total = this.can("voucher_read") ? res.data.total : 0;
                    this.loading = false;
                });
        },

        editItem(item) {
            this.editedIndex = this.vouchers.indexOf(item);
            this.editedItem = Object.assign({}, item);
            this.dialog = true;
        },
       	delteteSelectedRecords() {
            let just_ids = this.ids.map((e) => e.id);
            confirm(
                "Are you sure you wish to delete selected records , to mitigate any inconvenience in future."
            ) &&
                this.$axios
                    .post(this.endpoint + "-dsr", {
                        ids: just_ids,
                    })
                    .then((res) => {
                        if (!res.data.status) {
                            this.errors = res.data.errors;
                        } else {
                            this.$axios.get(this.endpoint).then((res) => {
                                this.vouchers = res.data.data;
                                this.total = res.data.total;
                                this.snackbar = true;
                                this.response.msg =
                                    "Selected records has been deleted";
								this.ids = [];
                            });
                        }
                    })
                    .catch((err) => console.log(err));
        },

        deleteItem(item) {
            confirm(
                "Are you sure you wish to delete , to mitigate any inconvenience in future."
            ) &&
                this.$axios
                    .delete(this.endpoint + "/" + item.id)
                    .then((res) => {
                        const index = this.vouchers.indexOf(item);
                        this.vouchers.splice(index, 1);
                        this.snackbar = res.data.status;
                        this.response.msg = res.data.message;
                    });
        },

        close() {
            this.dialog = false;
            setTimeout(() => {
                this.editedItem = Object.assign({}, this.defaultItem);
                this.editedIndex = -1;
            }, 300);
        },

        save() {
            let payload = {
				code: this.editedItem.code,
				expiry_date: this.editedItem.expiry_date,
				discount_value: this.editedItem.discount_value,
            };

            if (this.editedIndex > -1) {

				payload.status = "Used"
				
				this.$axios
                    .put(this.endpoint + "/" + this.editedItem.id, payload)
                    .then((res) => {
                        if (!res.data.status) {
                            this.errors = res.data.errors;
                        } else {
                            const index = this.vouchers.findIndex(
                                (item) => item.id == this.editedItem.id
                            );
                            this.vouchers.splice(index, 1, res.data.record);
                            this.snackbar = res.data.status;
                            this.response.msg = res.data.message;
                            this.close();
                        }
                    })
                    .catch((err) => console.log(err));
            } else {
                this.$axios
                    .post(this.endpoint, payload)
                    .then((res) => {
                        if (!res.data.status) {
                            this.errors = res.data.errors;
                        } else {
                            this.$axios
                                .get(this.endpoint + "?page=" + 1, {
                                    params: { per_page: 10 },
                                })
                                .then((res) => {
                                    this.vouchers = res.data.data;
                                    this.total = res.data.total;
                                    this.snackbar = res.data.status;
                                    this.response.msg = res.data.message;
                                    this.close();
                                });
                            this.errors = [];
                            this.search = "";
                        }
                    })
                    .catch((err) => console.log(err));
            }
        },
    },
};
</script>