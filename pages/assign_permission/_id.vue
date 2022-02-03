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
                            <div class="display-1 pa-2">Assign Permissions</div>
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
								v-for="(pa, idx) in permissions" 
								:key="pa.id" 
								:value="pa.id"
								v-model="permission_ids"
								:label="`${pa.permission}`"
								>
								</v-checkbox>
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
        role_id: "",
		permission_ids : [],
		permissions : [],
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
                this.roles = res.data.data.filter(e => e.role !== 'customer' || e.role !== 'master');
            })
            .catch((err) => console.log(err));

			this.$axios
            .get("permission-list")
            .then(({data}) => this.permissions = data)
            .catch((err) => console.log(err));

		this.$axios
            .get(`assign-permission/${this.$route.params.id}`)
            .then(({data}) => {
				this.role_id = data.role_id;
				this.permission_ids = data.permissions
            })
            .catch((err) => console.log(err));
    },
    methods: {

        save() {
            this.errors = [];
			let payload = {
				role_id : this.role_id,
				permissions : this.permission_ids 
			}

			this.$axios.put(`assign-permission/${this.$route.params.id}`, payload).then((res) => {
                if (res.data.status) {
                    this.msg = "Permissions assignement has been updated";
                    this.snackbar = res.data.status;
                    setTimeout(() => this.$router.push("/assign_permission"), 2000);
                } else {
                    this.errors = res.data.errors;
                }
            });
        },
    },
};
</script>