<template>
  <v-app>
    <v-main>
      <v-container class="fill-height">
        <v-row justify="center">
          <v-col cols="12" sm="8" md="6">
            <v-card class="elevation-12">
              <v-toolbar flat class="info" dark justify="center">
                <v-row justify="center" class="">
                  <v-col align="center">
                    <h2>Olvidé mi contraseña</h2>
                  </v-col>
                </v-row>
              </v-toolbar>
              <v-row justify="center" class="mt-5">
                <v-col align="center">
                  <NuxtLogo />
                </v-col>
              </v-row>
              <v-card-text>
                <v-form method="post" ref="form" lazy-validation>
                  <v-text-field
                    v-model="email"
                    label="Correo electrónico"
                    :rules="Rules"
                    required
                  >
                  </v-text-field>

                  <div
                    class="mb-3"
                    v-for="(error, index) in errors"
                    :key="index"
                  >
                    <span class="error--text">
                      {{ error }}
                    </span>
                  </div>

                  <div
                    class="mb-3"
                    style="width: 350px"
                    v-for="(message, index) in messages"
                    :key="index"
                  >
                    <span class="success--text">
                      {{ message }}
                    </span>
                  </div>

                  <v-row>
                    <v-col>
                      <v-btn
                        color="primary"
                        dark
                        :loading="loading"
                        @click="login"
                        >Enviar
                      </v-btn>
                    </v-col>
                    <v-col>
                      <div class="text-right mt-2">
                        <a href="/login">Haga clic aquí para iniciar sesión</a>
                      </div>
                    </v-col>
                  </v-row>
                </v-form>
              </v-card-text>

              <v-card-actions> </v-card-actions>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>
<script>
export default {
  auth: false,
  layout: "guest",
  data: () => ({
    messages: [],
    errors: [],
    loading: false,
    email: "",
    Rules: [(v) => !!v || "This field is required"],
  }),
  methods: {
    login() {
      if (this.$refs.form.validate()) {
        this.loading = true;

        try {
          this.$axios
            .post("reset_password", { email: this.email })
            .then((res) => {
              if (res.data.status) {
                this.messages = res.data.message;
                this.errors = [];
                this.loading = false;
              } else {
                this.messages = [];
                this.errors = res.data.errors;
                this.loading = true;
              }
            });
        } catch (error) {
          this.loading = false;
        }

        setTimeout(() => (this.loading = false), 2000);
      }
    },
  },
};
</script>