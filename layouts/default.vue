<template>
    <v-app dark>
        <v-navigation-drawer
            v-model="drawer"
            dark
            :mini-variant="miniVariant"
            :clipped="clipped"
            fixed
            app
            class="no_print"
            color="secondary"
        >
            <v-list
                v-for="(i, idx) in items"
                :key="idx"
                style="padding: 5px 0 0 0px"
            >
                <v-list-item
					
                    style="min-height: 0"
                    :to="i.to"
                    router
                    v-if="!i.hasChildren"
                >
                    <v-list-item-icon v-if="i.permission" class="ma-2">
                        <v-icon>{{ i.icon }}</v-icon>
                    </v-list-item-icon>
                    <v-list-item-title v-if="i.permission">{{ i.title }}</v-list-item-title>
                </v-list-item>

                <v-list-item
                    v-else
                    style="min-height: 0"
                    @click="i.open_menu = !i.open_menu"
                >
                    <v-list-item-icon v-if="i.permission" class="ma-2">
                        <v-icon>{{ i.icon }}</v-icon>
                    </v-list-item-icon>
                    <v-list-item-title v-if="i.permission">{{ i.title }}</v-list-item-title>
                    <v-icon v-if="i.permission" small>{{
                        !i.open_menu ? "mdi-chevron-down" : "mdi-chevron-up"
                    }}</v-icon>
                </v-list-item>
                <div v-if="i.open_menu">
                    <div
                        style="margin-left: 50px"
                        v-for="(j, jdx) in i.hasChildren"
                        :key="jdx"
                    >
                        <v-list-item 
						 v-if="j.permission"
						 style="min-height: 0" :to="j.to">
                            <v-list-item-title>{{ j.title }}</v-list-item-title>

                            <v-list-item-icon>
                                <v-icon :to="i.to">{{ j.icon }}</v-icon>
                            </v-list-item-icon>
                        </v-list-item>
                    </div>
                </div>
            </v-list>
        </v-navigation-drawer>
        <!-- color="#910105 #fd9d00" -->

        <v-app-bar
            color="primary"
            class="no_print"
            dark
            :clipped-left="clipped"
            fixed
            app
        >
            <v-app-bar-nav-icon @click.stop="drawer = !drawer" />
            {{ title }}
            <v-spacer />
            <span v-if="this.$auth.user">
                Welcome, <b>{{ this.$auth.user.name }}</b
                >&nbsp;
                <v-icon small @click="logout"> {{ logout_btn.icon }} </v-icon>
            </span>
        </v-app-bar>
        <v-main>
            <v-container>
                <nuxt />
            </v-container>
        </v-main>

        <v-footer :fixed="fixed" app>

			<v-icon Logout color="primary" @click="logout">{{logout_btn.icon}}</v-icon> 
        
				<span class="primary--text" >Logout</span>

				<v-spacer></v-spacer>
				<span class="primary--text">
					<v-icon v-if="$auth && this.$auth.user.role" Logout color="primary">mdi-account</v-icon> 
					{{Capitalize}}
				</span>	
			
        </v-footer>
    </v-app>
</template>

<script>
export default {
    mounted() {},
    data() {
        return {
            year: new Date().getFullYear(),
            clipped: false,
            open_menu: [],
            drawer: true,
            fixed: false,

            admins: [
                ["Management", "mdi-account-multiple-outline"],
                ["Settings", "mdi-cog-outline"],
            ],
            cruds: [
                ["Create", "mdi-plus-outline"],
                ["Read", "mdi-file-outline"],
                ["Update", "mdi-update"],
                ["Delete", "mdi-delete"],
            ],

            items: [
                {
                    icon: "mdi-home",
                    title: "Home",
                    to: "/",
					permission : this.can('/')
                },
                {
                    icon: "mdi-food",
                    title: "Kitchen",
                    to: "/kitchen",
					permission : this.can('kitchen_menu')
                },
                {
                    icon: "mdi-package-variant",
                    title: "Product Management",
                    to: "/product",
					permission : this.can('product_menu'),
					open_menu : false,
                    hasChildren: [
                        {
                            icon: "mdi-package-variant",
                            title: "Product",
                            to: "/product",
							permission : this.can('product_menu'),
                        },
						{
                            icon: "mdi-package-variant-closed",
                            title: "Category",
                            to: "/category",
							permission : this.can('category_menu'),
                        },
                        // {
                        //     icon: "mdi-package-variant-closed",
                        //     title: "Unit",
                        //     to: "/product_type",
                        // },
                    ],
                },
				{
                    icon: "mdi-account",
                    title: "User Management",
                    to: "/users",
					open_menu : false,
					permission : this.can('user_menu'),
                    hasChildren: [
                        {
                            icon: "mdi-account",
                            title: "Users",
                            to: "/users",
							permission : this.can('user_menu'),
                        },
                        {
                            icon: "mdi-account",
                            title: "Role",
                            to: "/role",
							permission : this.can('role_menu'),
                        },
						// {
                        //     icon: "mdi-account",
                        //     title: "Permission",
                        //     to: "/permission",
                        // },
						{
                            icon: "mdi-account",
                            title: "Permissions",
                            to: "/assign_permission",
							permission : this.can('assign_permission_menu'),							
                        },
                    ],
                },

                {
                    icon: "mdi-credit-card-outline",
                    title: "Payment Mode",
                    to: "/payment_mode",
					permission : this.can('payment_mode_menu'),							

                },
                {
                    icon: "mdi-credit-card-outline",
                    title: "Delivery Mode",
                    to: "/delivery_mode",
					permission : this.can('delivery_mode_menu'),							
                },
                {
                    icon: "mdi-credit-card-outline",
                    title: "Vouchers",
                    to: "/voucher",
					permission : this.can('voucher_menu'),							

                },
                {
                    icon: "mdi-city",
                    title: "Cities",
                    to: "/cities",
					permission : this.can('city_menu'),							
                },
                {
                    icon: "mdi-youtube",
                    title: "Promotional Video",
                    to: "/promotional_video",
					permission : this.can('promotional_video_menu'),							
                },
            ],

            miniVariant: false,
            title: "Food Portal Biryani",
            logout_btn: {
                icon: "mdi-logout",
                label: "Logout",
            },
        };
    },
    methods: {
		can(per) {
		let user = this.$auth.user;
		return (user && user.permissions.some(e => e.permission == per || per == '/') || user.master);
		},
        async logout() {
            await this.$auth.logout();
        },
    },
	computed: {
		Capitalize() {
			if(this.$auth && this.$auth.user.role){
				let role = this.$auth.user.role.role || '';
				return 'Logged in as ' + role.charAt(0).toUpperCase() + role.slice(1);
			}
			return '';
		}
	}
};
</script>
<style>
@media print {
    .no_print {
        display: none;
    }
}
/* div[data-app='true'] {
  background: url('file:///C:/Users/felix/Downloads/FINAL%20FOLDER%20OR%20UIZ%20FOR%20MINIMAX/FINAL%20FOLDER%20OR%20UIZ%20FOR%20MINIMAX/Website/FINAL%20HTML%20TEMPLATE%20FOR%20WEBSITE/HTML/assets/img/bgshapes.png') no-repeat center center fixed !important;
  background-size: cover;
} */
</style>
