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
            :items="users"
            :search="search"
            :options.sync="options"
            :server-items-length="total"
            :loading="loading"
            @pagination="paginate"
            :footer-props="{
                itemsPerPageOptions: [5, 10, 15],
            }"
            class="elevation-1"
        >
            <template v-slot:top>
                <v-toolbar flat>
                    <v-toolbar-title>Users</v-toolbar-title>
                    <v-divider class="mx-4" inset vertical></v-divider>

                    <v-text-field
                        v-model="search"
                        label="Search"
                        single-line
                        hide-details
                    ></v-text-field>
                    <v-divider class="mx-4" inset vertical></v-divider>
                    <v-btn
						v-if="can('user_delete')"
                        small
                        color="error"
                        class="mr-2 mb-2"
                        @click="delteteSelectedRecords"
                        >Delete Selected Records</v-btn
                    >
                    <v-btn
						v-if="can('user_created')"
                        small
                        class="mb-2 success accent--text"
                        to="/users/create"
                        >User +</v-btn
                    >
                    <v-dialog v-model="dialog" max-width="1000px">
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
                                        <v-row>
                                            <v-col cols="6">
                                                <v-text-field
                                                    :rules="Rules"
                                                    :readonly="isReadOnly"
                                                    v-model="editedItem.name"
                                                    label="User Name*"
                                                ></v-text-field>
                                            </v-col>

											  <v-col cols="6">
                                                <v-text-field
                                                    :rules="Rules"
                                                    :readonly="isReadOnly"
                                                    v-model="editedItem.email"
                                                    label="User Name*"
                                                ></v-text-field>
                                            </v-col>

											 <v-col cols="6">
                                                <v-text-field
                                                    :rules="Rules"
                                                    :readonly="isReadOnly"
                                                    v-model="editedItem.phone_number"
                                                    label="Phone Number*"
                                                ></v-text-field>
                                            </v-col>

											<v-col cols="6">
                                                <v-text-field
                                                    :rules="Rules"
                                                    :readonly="isReadOnly"
                                                    v-model="editedItem.cnic"
                                                    label="Cnic*"
                                                ></v-text-field>
                                            </v-col>

                                        </v-row>
                                        <v-row>
                                            <v-col>
                                                <v-text-field
                                                    :rules="Rules"
                                                    :readonly="isReadOnly"
                                                    v-model="
                                                        editedItem.address1
                                                    "
                                                    label="Location*"
                                                ></v-text-field>
                                            </v-col>
                                        </v-row>
                                      <v-row>
                                            <v-col>
                                                <v-text-field
													v-if="editedItem.role"
                                                    :rules="Rules"
                                                    :readonly="isReadOnly"
                                                    v-model="
                                                        editedItem.role.role
                                                    "
                                                    label="Role	*"
                                                ></v-text-field>
                                            </v-col>
                                        </v-row>
                                      
                                </v-container>
                            </v-card-text>
                        </v-card>
                    </v-dialog>
                </v-toolbar>
            </template>
            <template v-slot:item.action="{ item }">
					
                <v-icon v-if="can('user_read')" small color="info" class="mr-2" @click="viewItem(item)">
                    mdi-eye
                </v-icon>
                <v-icon
					v-if="can('user_edit')"
                    small
                    color="secondary"
                    class="mr-2"
                    @click="editItem(item)"
                >
                    mdi-pencil
                </v-icon>
                <v-icon
					v-if="can('user_delete')"
                    small
                    color="error"
                    class="mr-2"
                    @click="deleteItem(item)"
                >
                    mdi-delete
                </v-icon>
            </template>
        </v-data-table>
    </v-app>
</template>
<script>
export default {
    data: () => ({
        total: 0,
        per_page: 3,
        loading: true,
        options: {},
        search: "",
        isReadOnly: false,
        snackbar: false,
        dialog: false,
        ids: [],
        headers: [
            {
                text: "User Name",
                align: "left",
                sortable: false,
                value: "name",
            },
            {
                text: "Email",
                align: "left",
                sortable: false,
                value: "email",
            },
            {
                text: "Role",
                align: "left",
                sortable: false,
                value: "role.role",
            },
            { text: "Actions", value: "action", sortable: false },
        ],
        editedIndex: -1,
        editedItem: {
            name: "",
            location: "",
            lat: "",
            lon: "",
            description: "",
            streaming_url: "",
            number: "",
            city_id: "",
            IsActive: false,
        },
        defaultItem: {
            name: "",
            location: "",
            lat: "",
            lon: "",
            description: "",
            streaming_url: "",
            number: "",
            city_id: "",
            IsActive: false,
        },
        response: { msg: "" },
        users: [],
        cities: [],
		permissions_array : [],
		abilities : {
			create : "",
			read : "",
			update : "",
			delete : "",
		},
        Rules: [(v) => !!v || "This field is required"],
    }),

    computed: {
				
        formTitle() {
            return this.editedIndex === -1 ? "New Product" : "Edit Product";
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
        this.loading = false;		
    },

    methods: {
		can(permission) {
			let user = this.$auth.user;
			return user && user.permissions.some(e => e.permission == permission) || user.master;
		},
        async paginate(e) {
            let params = { params: { per_page: e.itemsPerPage } };

            this.$axios.get("users?page=" + e.page, params).then(({data}) => {
				this.users = this.can('user_read') ? data.data.filter(e => e.master == 0) : [];
				this.total = this.can('user_read') ? data.total : 0;
            });
        },
        async editItem(item) {
            this.$router.push(`/users/${item.id}`);
        },
        async viewItem(item) {
            this.editedIndex = this.users.indexOf(item);
            this.editedItem = Object.assign({}, item);
            this.editedItem.city_id = parseInt(item.city_id);
            this.editedItem.steaming_url = item.steaming_url || "Not Defined";
            this.dialog = true;
            this.isReadOnly = true;
        },

        delteteSelectedRecords() {
            let just_ids = this.ids.map((e) => e.id);
            confirm(
                "Are you sure you wish to delete selected records , to mitigate any inconvenience in future."
            ) &&
                this.$axios
                    .post("users-dsr", {
                        ids: just_ids,
                    })
                    .then((res) => {
                        if (!res.data.status) {
                            this.errors = res.data.errors;
                        } else {
                            this.$axios.get("users").then((res) => {
                                this.users = res.data.data;
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
                this.$axios.delete("users/" + item.id).then((res) => {
                    const index = this.users.indexOf(item);
                    this.users.splice(index, 1);
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
            this.editedItem.name = this.editedItem.name.toLowerCase();
            this.$axios
                .put("users/" + this.editedItem.id, this.editedItem)
                .then((res) => {
                    if (!res.data.status) {
                        this.errors = res.data.errors;
                    } else {
                        const idx = this.users.findIndex(
                            (item) => item.id == this.editedItem.id
                        );
                        Object.assign(this.users[idx], res.data.record);
                        this.snackbar = res.data.status;
                        this.response.msg = res.data.message;
                        this.close();
                    }
                });
        },
    },
};
</script>