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
            :items="payment_modes"
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
                    <v-toolbar-title>Payment Mode</v-toolbar-title>
                    <v-divider class="mx-4" inset vertical></v-divider>

                    <v-text-field
                        v-model="search"
                        label="Search"
                        single-line
                        hide-details
                    ></v-text-field>
                    <v-divider class="mx-4" inset vertical></v-divider>
                    <v-btn
                        v-if="can('payment_mode_delete')"
                        small
                        color="error"
                        class="mr-2 mb-2"
                        @click="delteteSelectedRecords"
                        >Delete Selected Records</v-btn
                    >
                    <v-btn
                        v-if="can('payment_mode_create')"
                        small
                        color="success"
                        class="mb-2"
                        @click="dialog = true"
                        >Payment Mode +</v-btn
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
                                                v-model="editedItem.title"
                                                label="Title"
                                            ></v-text-field>
                                            <span
                                                v-if="
                                                    errors && errors.length > 0
                                                "
                                                class="error--text"
                                                >{{ errors[0] }}</span
                                            >
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
                    v-if="can('payment_mode_edit')"
                    color="secondary"
                    small
                    class="mr-2"
                    @click="editItem(item)"
                >
                    mdi-pencil
                </v-icon>
                <v-icon
                    v-if="can('payment_mode_delete')"
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
        search: "",
        endpoint: "payment-mode",
        snackbar: false,
        dialog: false,
        ids: [],
        total: 0,
        loading: false,
        headers: [
            {
                text: "Payment Mode",
                align: "left",
                sortable: false,
                value: "title",
            },
            { text: "Actions", value: "action", sortable: false },
        ],
        editedIndex: -1,
        editedItem: { title: "" },
        defaultItem: { title: "" },
        response: { msg: "" },
        payment_modes: [],
        errors: [],
        params: {},
    }),

    computed: {
        formTitle() {
            return this.editedIndex === -1
                ? "New Payment Mode"
                : "Edit Payment Mode";
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
                    this.payment_modes = this.can("payment_mode_read")
                        ? res.data.data
                        : [];
                    this.total = this.can("payment_mode_read")
                        ? res.data.total
                        : 0;
                    this.loading = false;
                });
        },

        editItem(item) {
            this.editedIndex = this.payment_modes.indexOf(item);
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
                            this.$axios
                                .get(this.endpoint + "?page=" + 1, {
                                    params: { per_page: 10 },
                                })
                                .then((res) => {
                                    this.payment_modes = res.data.data;
                                    this.total = res.data.total;
                                    this.snackbar = res.data.status;
                                    this.ids = [];
                                    this.response.msg =
                                        "Selected records has been deleted";
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
                        const index = this.payment_modes.indexOf(item);
                        this.payment_modes.splice(index, 1);
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
                title: this.editedItem.title.toLowerCase(),
            };

            if (this.editedIndex > -1) {
                this.$axios
                    .put(this.endpoint + "/" + this.editedItem.id, payload)
                    .then((res) => {
                        if (!res.data.status) {
                            this.errors = res.data.errors;
                        } else {
                            const index = this.payment_modes.findIndex(
                                (item) => item.id == this.editedItem.id
                            );
                            this.payment_modes.splice(
                                index,
                                1,
                                res.data.record
                            );
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
                                    this.payment_modes = res.data.data;
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