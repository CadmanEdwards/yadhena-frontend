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
            :items="kitchens"
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
                    <v-toolbar-title>Kitchens</v-toolbar-title>
                    <v-divider class="mx-4" inset vertical></v-divider>

                    <v-text-field
                        v-model="search"
                        label="Search"
                        single-line
                        hide-details
                    ></v-text-field>
                    <v-divider class="mx-4" inset vertical></v-divider>
                    <v-btn
						v-if="can('kitchen_delete')"
                        small
                        color="error"
                        class="mr-2 mb-2"
                        @click="delteteSelectedRecords"
                        >Delete Selected Records</v-btn
                    >
                    <v-btn
                        v-if="can('kitchen_create')"
                        small
                        class="mb-2 success accent--text"
                        to="/kitchen/create"
                        >Kitchen +</v-btn
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
                                    <v-form ref="form" lazy-validation>
                                        <v-row>
                                            <v-col cols="6">
                                                <v-text-field
                                                    :rules="Rules"
                                                    :readonly="isReadOnly"
                                                    v-model="editedItem.name"
                                                    label="Kitchen Name*"
                                                ></v-text-field>
                                            </v-col>

                                            <v-col cols="6">
                                                <v-select
                                                    :readonly="isReadOnly"
                                                    v-model="editedItem.city_id"
                                                    :items="cities"
                                                    item-text="city"
                                                    item-value="id"
                                                    label="City*"
                                                    persistent-hint
                                                ></v-select>
                                            </v-col>
                                            <v-col cols="6">
                                                <v-text-field
                                                    type="number"
                                                    :rules="Rules"
                                                    v-model="editedItem.lat"
                                                    label="Lat"
                                                ></v-text-field>
                                            </v-col>
                                            <v-col cols="6">
                                                <v-text-field
                                                    type="number"
                                                    :rules="Rules"
                                                    v-model="editedItem.lon"
                                                    label="Lon"
                                                ></v-text-field>
                                            </v-col>
                                        </v-row>

                                        <v-row>
                                            <v-col>
                                                <v-text-field
                                                    :rules="Rules"
                                                    :readonly="isReadOnly"
                                                    v-model="editedItem.number"
                                                    label="Number"
                                                ></v-text-field>
                                            </v-col>
                                            <v-col>
                                                <v-text-field
                                                    :rules="Rules"
                                                    :readonly="isReadOnly"
                                                    v-model="
                                                        editedItem.steaming_url
                                                    "
                                                    label="Steaming Url"
                                                ></v-text-field>
                                            </v-col>
                                        </v-row>

                                        <v-row>
                                            <v-col>
                                                <v-text-field
                                                    :rules="Rules"
                                                    :readonly="isReadOnly"
                                                    v-model="
                                                        editedItem.location
                                                    "
                                                    label="Location*"
                                                ></v-text-field>
                                            </v-col>
                                        </v-row>
                                        <v-row>
                                            <v-col>
                                                <v-text-field
                                                    :rules="Rules"
                                                    :readonly="isReadOnly"
                                                    v-model="
                                                        editedItem.description
                                                    "
                                                    label="Description"
                                                ></v-text-field>
                                            </v-col>
                                        </v-row>
                                        <v-row v-if="!isReadOnly">
                                            <v-col>
                                                <v-checkbox
                                                    color="primary"
                                                    v-model="
                                                        editedItem.IsActive
                                                    "
                                                    label="Active"
                                                ></v-checkbox>
                                            </v-col>
                                        </v-row>
                                        <v-row>
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
                <v-icon
                    v-if="can('kitchen_read')"
                    small
                    color="info"
                    class="mr-2"
                    @click="viewItem(item)"
                >
                    mdi-eye
                </v-icon>
                <v-icon
					v-if="can('kitchen_edit')"
                    small
                    color="secondary"
                    class="mr-2"
                    @click="editItem(item)"
                >
                    mdi-pencil
                </v-icon>
                <v-icon
					v-if="can('kitchen_delete')"
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
                text: "Kitchen Name",
                align: "left",
                sortable: false,
                value: "name",
            },
            {
                text: "City",
                align: "left",
                sortable: false,
                value: "city.city",
            },
            {
                text: "Location",
                align: "left",
                sortable: false,
                value: "location",
            },
            {
                text: "Number",
                align: "left",
                sortable: false,
                value: "number",
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
        kitchens: [],
        cities: [],
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
        const cities = await this.$axios.get("city");
        this.cities = cities.data.data;
        this.loading = false;
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
            let params = { params: { per_page: e.itemsPerPage } };

            this.$axios.get("kitchen?page=" + e.page, params).then((res) => {
				this.kitchens = this.can('kitchen_read') ? res.data.data : [];
                this.total = this.can('kitchen_read') ? res.data.total : 0;
            });
        },
        async editItem(item) {
            this.getSpecificIndex(item, false);
        },
        async viewItem(item) {
            this.getSpecificIndex(item, true);
        },

        getSpecificIndex(item, condition) {
            this.editedIndex = this.kitchens.indexOf(item);
            this.editedItem = Object.assign({}, item);
            this.editedItem.city_id = parseInt(item.city_id);
            this.editedItem.steaming_url = item.steaming_url || "Not Defined";
            this.dialog = true;
            this.isReadOnly = condition;
        },

        delteteSelectedRecords() {
            let just_ids = this.ids.map((e) => e.id);
            confirm(
                "Are you sure you wish to delete selected records , to mitigate any inconvenience in future."
            ) &&
                this.$axios
                    .post("kitchen-dsr", {
                        ids: just_ids,
                    })
                    .then((res) => {
                        if (!res.data.status) {
                            this.errors = res.data.errors;
                        } else {
                            this.$axios.get("kitchen").then((res) => {
                                this.kitchens = res.data.data;
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
                this.$axios.delete("kitchen/" + item.id).then((res) => {
                    const index = this.kitchens.indexOf(item);
                    this.kitchens.splice(index, 1);
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
                .put("kitchen/" + this.editedItem.id, this.editedItem)
                .then((res) => {
                    if (!res.data.status) {
                        this.errors = res.data.errors;
                    } else {
                        const idx = this.kitchens.findIndex(
                            (item) => item.id == this.editedItem.id
                        );
                        Object.assign(this.kitchens[idx], res.data.record);
                        this.snackbar = res.data.status;
                        this.response.msg = res.data.message;
                        this.close();
                    }
                });
        },
    },
};
</script>