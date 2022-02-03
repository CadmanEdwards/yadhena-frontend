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
                            <div class="display-1 pa-2">Add Kitchen</div>
                        </v-col>
						<v-col>
                            <div class="display-1 pa-2  text-right">
								<v-btn class="primary" to="/kitchen">
									<v-icon>mdi-arrow-left</v-icon>&nbsp;Back
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
                                    label="Kitchen Location Name*"
                                ></v-text-field>
                            </v-col>
                            <v-col cols="6">
                                <v-select
                                    :rules="Rules"
                                    v-model="city_id"
                                    :items="cities"
                                    item-value="id"
                                    item-text="city"
                                    label="City*"
                                ></v-select>
                            </v-col>

                            <v-col cols="6">
                                <v-text-field
                                    type="number"
                                    :rules="Rules"
                                    v-model="lat"
                                    label="Lat"
                                ></v-text-field>
                            </v-col>
                            <v-col cols="6">
                                <v-text-field
                                    type="number"
                                    :rules="Rules"
                                    v-model="lon"
                                    label="Lon"
                                ></v-text-field>
                            </v-col>

                            <v-col cols="6">
                                <v-text-field
                                    type="number"
                                    v-model="number"
                                    label="Mobile Number"
                                ></v-text-field>
                            </v-col>
                            <v-col cols="6">
                                <v-text-field
                                    type="url"
                                    v-model="streaming_url"
                                    label="Streaming URL"
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
                                <v-textarea
                                    v-model="description"
                                    label="Description"
                                ></v-textarea>
                            </v-col>
                        </v-row>
                        <v-row>
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
        cities: [],
        name: "",
        location: "",
        lat: "",
        lon: "",
        city_id: "",
        description: "",
        IsActive: "",
        streaming_url: "",
        number: "",
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
            .get("city")
            .then((res) => {
                this.cities = res.data.data;
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
            this.errors = [];
            let kitchen = new FormData();
            kitchen.append("name", this.name.toLowerCase());
            kitchen.append("location", this.location);
            kitchen.append("lat", this.lat);
            kitchen.append("lon", this.lon);
            kitchen.append("city_id", this.city_id);
            kitchen.append("description", this.description);
            kitchen.append("streaming_url", this.streaming_url);
            kitchen.append("number", this.number);
            kitchen.append("IsActive", this.IsActive == true ? 1 : 0);

            this.$axios.post("kitchen", kitchen).then((res) => {
                if (res.data.status) {
                    this.msg = "Kitchen has been added";
                    this.snackbar = res.data.status;
                    setTimeout(() => this.$router.push("/kitchen"), 2000);
                } else {
                    this.errors = res.data.errors;
                }
            });
        },
    },
};
</script>