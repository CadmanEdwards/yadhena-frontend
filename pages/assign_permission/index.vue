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
            :items="permissions"
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
                    <v-toolbar-title>Assigned Permissions</v-toolbar-title>
                    <v-divider class="mx-4" inset vertical></v-divider>

                    <v-text-field
                        v-model="search"
                        label="Search"
                        single-line
                        hide-details
                    ></v-text-field>
                    <v-divider class="mx-4" inset vertical></v-divider>
                    <v-btn
                        v-if="can('assign_permission_delete')"
                        small
                        color="error"
                        class="mr-2 mb-2"
                        @click="delteteSelectedRecords"
                        >Delete Selected Records</v-btn
                    >
                    <v-btn
                        v-if="can('assign_permission_create')"
                        small
                        class="mb-2 success accent--text"
                        to="/assign_permission/create"
                        >Assign Permission +</v-btn
                    >
                </v-toolbar>
            </template>
            <template v-slot:item.permissions_array="{ item }">
                <v-chip
                    class="ma-1"
                    small
                    color="primary"
                    v-for="(pa, idx) in item.permissions_array"
                    :key="pa.id"
                >
                    {{ pa.permission }}
                </v-chip>
            </template>

            <template v-slot:item.action="{ item }">
                <v-icon
                    color="secondary"
                    small
                    class="mr-2"
                    @click="editItem(item)"
                >
                    mdi-pencil
                </v-icon>
                <v-icon
                    v-if="can('assign_permission_delete')"
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
                text: "Role",
                align: "left",
                sortable: false,
                value: "role.role",
            },
            {
                text: "Permissions",
                align: "left",
                sortable: false,
                value: "permissions_array",
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
        permissions: [],
        response: { msg: "" },
    }),
    async created() {
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

            this.$axios
                .get("assign-permission?page=" + e.page, params)
                .then((res) => {
					this.permissions = this.can('assign_permission_read') ? res.data.data : [];
					this.total = this.can('assign_permission_read') ? res.data.total : 0;
                });
        },
        async editItem(item) {
            this.$router.push(`/assign_permission/${item.id}`);
        },
        async viewItem(item) {
            this.editedIndex = this.permissions.indexOf(item);
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
                    .post("assign-permission-dsr", {
                        ids: just_ids,
                    })
                    .then((res) => {
                        if (!res.data.status) {
                            this.errors = res.data.errors;
                        } else {
                            this.$axios.get("assign-permission").then((res) => {
                                this.permissions = res.data.data;
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
                this.$axios
                    .delete("assign-permission/" + item.id)
                    .then((res) => {
                        const index = this.permissions.indexOf(item);
                        this.permissions.splice(index, 1);
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
                .put("assign-permission/" + this.editedItem.id, this.editedItem)
                .then((res) => {
                    if (!res.data.status) {
                        this.errors = res.data.errors;
                    } else {
                        const idx = this.permissions.findIndex(
                            (item) => item.id == this.editedItem.id
                        );
                        Object.assign(this.permissions[idx], res.data.record);
                        this.snackbar = res.data.status;
                        this.response.msg = res.data.message;
                        this.close();
                    }
                });
        },
    },
};
</script>