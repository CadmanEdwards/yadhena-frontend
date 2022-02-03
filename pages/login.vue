<template>
    <v-app>
        <section class="h-100 gradient-form" style="background-color: #eee">
            <div class="container py-5 h-100">
                <div
                    class="
                        row
                        d-flex
                        justify-content-center
                        align-items-center
                        h-100
                    "
                >
                    <div class="col-xl-10">
                        <div class="card rounded-3 text-black">
                            <div class="row g-0">
                                <div class="col-lg-6">
                                    <div class="card-body p-md-5 mx-md-4">
                                        <div class="text-center">
                                            <img
                                                src="/LTLOGO.png"
                                                style="width: 185px"
                                                alt="logo"
                                            />
                                            <h4 class="mt-1 mb-5 pb-1">
                                                ADMIN PANEL
                                            </h4>
                                        </div>

                                        <v-form ref="form" method="post">
                                            <p>Please login to your account</p>
                                            <label for="">Email</label>
                                            <div class="form-outline mb-4">
                                                <input
                                                    v-model="email"
                                                    style="border: 1px solid"
                                                    type="email"
                                                    id="form2Example11"
                                                    class="form-control"
                                                    placeholder="admin@example.com"
                                                />
                                            </div>

                                            <label for="">Password</label>

                                            <div class="form-outline mb-4">
                                                <input
                                                    v-model="password"
                                                    style="border: 1px solid"
                                                    type="password"
                                                    id="form2Example22"
                                                    class="form-control"
                                                    placeholder="secret"

                                                />
                                            </div>

                                            <div
                                                class="
                                                    text-center
                                                    pt-1
                                                    mb-5
                                                    pb-1
                                                "
                                            >
                                                <span
                                                    v-if="msg"
                                                    class="error--text"
                                                    >{{ msg }}</span
                                                >
                                                <v-btn
                                                    :loading="loading"
                                                    @click="login"
                                                    class="
                                                        btn
                                                        btn-primary
                                                        btn-block
                                                        text-white
                                                        fa-lg
                                                        gradient-custom-2
                                                        mt-1
                                                        mb-3
                                                    "
                                                >
                                                    Log in
                                                </v-btn>

                                                <!-- <a class="text-muted" href="#!">Forgot password?</a> -->
                                            </div>

                                            <div
                                                class="
                                                    d-flex
                                                    align-items-center
                                                    justify-content-center
                                                    pb-4
                                                "
                                            >
                                                <!-- <p class="mb-0 me-2">Don't have an account?</p> -->
                                                <!-- <button type="button" class="btn btn-outline-danger">Create new</button> -->
                                            </div>
                                        </v-form>
                                    </div>
                                </div>
                                <div
                                    class="
                                        col-lg-6
                                        d-flex
                                        align-items-center
                                        gradient-custom-2
                                    "
                                >
                                    <div
                                        class="
                                            text-white
                                            px-3
                                            py-4
                                            p-md-5
                                            mx-md-4
                                        "
                                    >
                                        <h4 class="mb-4">
											We are more than just a food company
                                        </h4>
                                        <p class="small mb-0">
                                            We don't have a complex story. We always had passion to share great food with the audience but, then we thought why not to share our fortune with our valuable clients. We started Lt Biryani by opening kitchens in five different cities of Pakistan and plans to open more in future.
                                        </p>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </section>
    </v-app>
</template>

<script>
export default {
    layout: "login",
    data: () => ({
        loading: false,
        snackbar: false,
        email: "",
        password: "",
        msg: "",
    }),
    methods: {
        login() {
            this.msg = "";
            this.loading = true;

            let credentials = {
                email: this.email,
                password: this.password,
            };
            this.$auth.loginWith("local", { data: credentials }).catch((e) => {
                this.msg = e.response.data.error;
                setTimeout(() => (this.loading = false), 2000);
            });
        },
    },
};
</script>
<style >
.gradient-custom-2 {
    /* fallback for old browsers */
    background: #fccb90;

    /* Chrome 10-25, Safari 5.1-6 */
    background: -webkit-linear-gradient(
        to right,
        #ee7724,
        #d8363a,
        #dd3675,
        #b44593
    );

    /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
    background: linear-gradient(to right, #ee7724, #d8363a, #dd3675, #b44593);
}

@media (min-width: 768px) {
    .gradient-form {
        height: 100vh !important;
    }
}
@media (min-width: 769px) {
    .gradient-custom-2 {
        border-top-right-radius: 0.3rem;
        border-bottom-right-radius: 0.3rem;
    }
}
</style>
