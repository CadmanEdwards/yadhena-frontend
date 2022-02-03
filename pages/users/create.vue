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
                            <div class="display-1 pa-2">Add User</div>
                        </v-col>
                        <v-col>
                            <div class="display-1 pa-2 text-right">
                                <v-btn small class="primary" to="/users">
                                    <v-icon small>mdi-arrow-left</v-icon
                                    >&nbsp;Back
                                </v-btn>
                            </div>
                        </v-col>
                    </v-row>
                    <v-container>
                        <v-row>
                            <v-col cols="6">
                                <v-text-field
                                    :rules="Rules"
                                    v-model="name"
                                    label="User Name*"
                                ></v-text-field>
                            </v-col>
                            <v-col cols="6">
                                <v-text-field
                                    :rules="Rules"
                                    v-model="email"
                                    label="User Email*"
                                ></v-text-field>
                            </v-col>
                        </v-row>
                        <v-row>
                            <v-col cols="6">
                                <v-text-field
                                    type="password"
                                    :rules="Rules"
                                    v-model="password"
                                    label="Password*"
                                ></v-text-field>
                            </v-col>
                            <v-col cols="6">
                                <v-text-field
                                    type="password"
                                    :rules="Rules"
                                    v-model="confirm_password"
                                    label="Confirm Password*"
                                ></v-text-field>
                            </v-col>
                        </v-row>
                        <v-row>
                            <v-col cols="6">
                                <v-text-field
                                    type="number"
                                    v-model="phone_number"
                                    label="Mobile Number"
                                ></v-text-field>
                            </v-col>
                            <v-col cols="6">
                                <v-text-field
                                    type="number"
                                    v-model="cnic"
                                    label="Cnic"
                                ></v-text-field>
                            </v-col>
                        </v-row>

                        <v-row>
                            <v-col cols="12">
                                <v-textarea
                                    v-model="location"
                                    label="Location*"
                                ></v-textarea>
                            </v-col>
                        </v-row>

                        <v-row>
                            <v-col cols="12">
                                <v-select
                                    :rules="Rules"
                                    v-model="role_id"
                                    :items="roles"
                                    item-value="id"
                                    item-text="role"
                                    label="Role*"
                                ></v-select>
                            </v-col>

                            <v-col cols="12">
                                <v-checkbox
                                    color="primary"
                                    v-model="IsActive"
                                    label="Active"
                                ></v-checkbox>
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



<script>
export default {
    data: () => ({
        name: "",
        email: "",
        password: "",
        confirm_password: "",
        cnic: "",
        location: "",
        role_id: "",
        IsActive: "",
        phone_number: "",
        msg: "",
        snackbar: false,
        Rules: [(v) => !!v || "This field is required"],
        errors: [],
        roles: [],
    }),
    created() {
        this.$axios
            .get("role")
            .then((res) => {
                this.roles = res.data.data.filter(e => e.role !== 'customer');
            })
            .catch((err) => console.log(err));
    },
    methods: {

        save() {
            this.errors = [];
            let user = new FormData();
            user.append("name", this.name.toLowerCase());
            user.append("email", this.email);
            user.append("password", this.password);
            user.append("confirm_password", this.confirm_password);
            user.append("phone_number", this.phone_number);
            user.append("cnic", this.cnic);
            user.append("location", this.location);
            user.append("role_id", this.role_id);
            user.append("IsActive", this.IsActive == true ? 1 : 0);

            this.$axios.post("users", user).then((res) => {
                if (res.data.status) {
                    this.msg = "User has been added";
                    this.snackbar = res.data.status;
                    setTimeout(() => this.$router.push("/users"), 2000);
                } else {
                    this.errors = res.data.errors;
                }
            });
        },
    },
};
</script>