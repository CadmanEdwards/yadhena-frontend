<template>
    <div>
        <v-row>
		    <v-col cols="9">
                <v-row>
                    <v-col v-for="i in total_items" :key="i" cols="4">
                        <v-card class="no_print">
                            <v-list-item four-line>
                                <v-list-item-content>
                                    <div
                                        class="
                                            heading
                                            display-3
                                            overline
                                            mb-4
                                            text-center
                                        "
                                    >
                                        <b>{{ i.title }}</b>
                                    </div>
                                    <div
                                        class="
                                            display-1
                                            mb-4
                                            primary--text
                                            text-center
                                        "
                                    >
                                        <b>{{ i.value }}</b>
                                    </div>
                                    <v-list-item-subtitle>
                                    </v-list-item-subtitle>
                                </v-list-item-content>
                            </v-list-item>
                        </v-card>
                    </v-col>
                </v-row>
                <v-row>
                    <v-col v-for="i in items_by_cities" :key="i" cols="4">
                        <v-card class="no_print" height="150px">
                            <v-list-item four-line>
                                <v-list-item-content>
                                    <div
                                        class="
                                            heading
                                            display-3
                                            overline
                                            mb-4
                                            text-center
                                        "
                                    >
                                        <b>{{ i.title }}</b>
                                    </div>
                                    <div
                                        class="
                                            display-1
                                            mb-4
                                            primary--text
                                            text-center
                                        "
                                    >
                                        <b>{{ i.value }}</b>
                                    </div>
                                    <v-list-item-subtitle>
                                    </v-list-item-subtitle>
                                </v-list-item-content>
                            </v-list-item>
                        </v-card>
                    </v-col>
                </v-row>

                <v-row>
                    <v-col cols="6">
                        <v-card class="no_print">
                            <v-list-item four-line>
                                <v-list-item-content>
                                    <div
                                        class="
                                            heading
                                            display-3
                                            overline
                                            mb-4
                                            text-center
                                        "
                                    >
                                        <b>PAYMENT MODES</b>
                                    </div>

                                    <v-list-item-subtitle
                                        v-for="(i, idx) in payment_modes"
                                        :key="idx"
                                    >
                                        <div>
                                            {{ i.title }}
                                            <v-progress-linear
                                                :color="i.color"
                                                class="mt-2"
                                                height="15"
                                                striped
                                                :value="i.value"
                                            >
                                                <strong
                                                    >{{
                                                        Math.ceil(i.value)
                                                    }}%</strong
                                                >
                                            </v-progress-linear>
                                        </div>
                                    </v-list-item-subtitle>
                                </v-list-item-content>
                            </v-list-item>
                        </v-card>
                    </v-col>

                    <v-col cols="6">
                        <v-row>
                            <v-col cols="12">
                                <v-card class="no_print">
                                    <v-list-item four-line>
                                        <v-list-item-content>
                                            <div
                                                class="
                                                    heading
                                                    display-3
                                                    overline
                                                    mb-4
                                                    text-center
                                                "
                                            >
                                                <b>ORDERS BY DEVICE</b>
                                            </div>

                                            <v-list-item-subtitle
                                                v-for="(
                                                    i, idx
                                                ) in order_by_devices"
                                                :key="idx"
                                            >
                                                <div>
                                                    {{ i.title }}
                                                    <v-progress-linear
                                                        :color="i.color"
                                                        class="mt-2"
                                                        height="15"
                                                        rounded
                                                        :value="i.value"
                                                    >
                                                        <strong
                                                            >{{
                                                                Math.ceil(
                                                                    i.value
                                                                )
                                                            }}%</strong
                                                        >
                                                    </v-progress-linear>
                                                </div>
                                            </v-list-item-subtitle>
                                        </v-list-item-content>
                                    </v-list-item>
                                </v-card>
                            </v-col>

                            <!-- <v-col cols="12">
                                <v-card class="no_print">
                                    <v-list-item four-line>
                                        <v-list-item-content>
                                            <div
                                                class="
                                                    heading
                                                    display-3
                                                    overline
                                                    mb-4
                                                    text-center
                                                "
                                            >
                                                <b>ACTIVE ORDERS</b>
                                            </div>

                                            <div
                                                class="
                                                    display-1
                                                    mb-4
                                                    primary--text
                                                    text-center
                                                "
                                            >
                                                <b>25</b>
                                            </div>
                                        </v-list-item-content>
                                    </v-list-item>
                                </v-card>
                            </v-col> -->
                        </v-row>
                    </v-col>
                </v-row>
            </v-col>
            <v-col cols="3">
                <v-row>
                    <v-col>
                        <v-card class="no_print">
                            <v-col cols="12">
                                <v-list-item four-line>
                                    <v-list-item-content>
                                        <div
                                            class="
                                                heading
                                                display-3
                                                overline
                                                mb-4
                                                text-center
                                            "
                                        >
                                            TODAY'S CITYWISE ORDER COUNT
                                        </div>
                                        <!-- <div class="display-1 mb-4 text-center">
                                            <b>45.5K</b>
                                        </div> -->

                                        <v-col
                                            v-for="(i, idx) in items_by_cities"
                                            :key="idx"
                                            cols="12"
                                        >
                                            {{ i.title }}

                                            <div
                                                style="margin-top: -10%"
                                                class="text-right"
                                            >
                                                <v-chip small color="error">{{
                                                    i.value
                                                }}</v-chip>
                                            </div>
                                        </v-col>
                                    </v-list-item-content>
                                </v-list-item>
                            </v-col>
                        </v-card>
                        <v-card class="mt-2 no_print primary" dark>
                            <v-list-item four-line>
                                <v-list-item-content>
                                    <div
                                        class="
                                            heading
                                            display-3
                                            overline
                                            mb-4
                                            text-center
                                        "
                                    >
                                        <b>ACTIVE ORDERS</b>
                                    </div>
                                    <div class="display-1 mb-4 text-center">
                                        <b>25</b>
                                    </div>
                                    <v-list-item-subtitle>
                                    </v-list-item-subtitle>
                                </v-list-item-content>
                            </v-list-item>
                        </v-card>
                        <v-card class="mt-2 no_print primary" dark>
                            <v-list-item four-line>
                                <v-list-item-content>
                                    <div
                                        class="
                                            heading
                                            display-3
                                            overline
                                            mb-4
                                            text-center
                                        "
                                    >
                                        <b>Total Employees</b>
                                    </div>
                                    <div class="display-1 mb-4 text-center">
                                        <b>51</b>
                                    </div>
                                    <v-list-item-subtitle>
                                    </v-list-item-subtitle>
                                </v-list-item-content>
                            </v-list-item>
                        </v-card>
                    </v-col>
                </v-row>
            </v-col>
        </v-row>
    </div>
</template>

<script>
import RecentOrder from "../components/Order/Recent_Order";

export default {
    components: {
        RecentOrder,
    },

    data() {
        return {
            total_items: [],
            items_by_cities: [],
            test_headers: [
                {
                    text: "Customer",
                    align: "left",
                    sortable: false,
                    value: "company_name",
                },
                {
                    text: "Order Total",
                    align: "left",
                    sortable: false,
                    value: "order_total",
                },
            ],

            orders: "",
            products: "",
            customers: "",
            daily_orders: "",
            weekly_orders: "",
            monthly_orders: "",
        };
    },
    created() {
        this.initialize();
    },
    filters: {
        get_decimal_value: function (value) {
            if (!value) return "";
            return (Math.round(value * 100) / 100).toFixed(2);
        },
        get_comma_seperator: function (x) {
            if (!x) return "";
            return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
        },
    },
    methods: {
        async initialize() {
            (this.total_items = [
                {
                    title: "TOTAL ORDERS",
                    value: "45.5k",
                },
                {
                    title: "TOTAL POS ORDERS",
                    value: "5.2k",
                },
                {
                    title: "TOTAL DIGITAL ORDERS",
                    value: "4.2k",
                },
            ]),
                (this.items_by_cities = [
                    {
                        title: "ISLAMABD",
                        value: "45.5k",
                    },
                    {
                        title: "KARACHI",
                        value: "5.2k",
                    },
                    {
                        title: "LAHORE",
                        value: "4.2k",
                    },
                    {
                        title: "FAISALABD",
                        value: "4.2k",
                    },
                    {
                        title: "SIALKOT",
                        value: "4.2k",
                    },
                    {
                        title: "RAWALPINDI",
                        value: "4.2k",
                    },
                ]),
                (this.payment_modes = [
                    {
                        title: "CASH ON DELIVER",
                        value: 9.5,
                        color: "success",
                    },
                    {
                        title: "EASY PAISA",
                        value: 3.2,
                        color: "warning",
                    },
                    {
                        title: "JAZZ CASH",
                        value: 7.2,
                        color: "info",
                    },
                    {
                        title: "DEBIT CARD",
                        value: 1.2,
                        color: "error",
                    },
                ]);
            this.order_by_devices = [
                {
                    title: "DESKTOP",
                    value: 3.2,
                    color: "success",
                },
                {
                    title: "MOBILE",
                    value: 7.2,
                    color: "warning",
                },
                {
                    title: "POS",
                    value: 1.2,
                    color: "error",
                },
            ];
        },
    },
};
</script>