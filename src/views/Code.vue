<template>
  <div>
    <div>{{ query }}</div>
    <button @click="getTokens" class="button">Get Tokens</button>
    <div v-if="idToken.length > 0">
      <label>ID Token</label>
      <div class="token">{{ idToken }}</div>
      <label>Access Token</label>
      <div class="token">{{ accessToken }}</div>
      <button class="button" @click="callApi()">Call API</button>
    </div>
  </div>
</template>

<script>
const qs = require("querystring");
const pool = require("@/assets/pool-config.js");
const axios = require("axios");

export default {
  props: ["query"],
  data() {
    return {
      accessToken: "",
      idToken: "",
      decodedIdToken: undefined,
      decodedAccessToken: undefined,
    };
  },
  methods: {
    getTokens() {
      axios
        .post(
          `${pool.HOSTED_UI}/oauth2/token`,
          qs.stringify({
            grant_type: "authorization_code",
            client_id: pool.CLIENTID,
            scope: "openid",
            code: this.query.code,
            redirect_uri: pool.HTTPS_REDIRECT_GW_URI,
          }),
          {
            headers: {
              "Content-Type": "application/x-www-form-urlencoded",
            },
          }
        )
        .then((res) => {
          this.accessToken = res.data.access_token;
          this.idToken = res.data.id_token;
        })
        .catch((error) => {
          // eslint-disable-next-line no-console
          console.error("Some bug information", error);
        });
    },
    callApi() {
      const api_gateway_url =
        "https://xxxxxxxxxxx.execute-api.eu-west-1.amazonaws.com/demo/";
      axios
        .get(api_gateway_url, {
          headers: { Authorization: this.idToken },
        })
        .then((res) => {
          // eslint-disable-next-line no-console
          console.log(res.data);
        })
        .catch((err) => {
          // eslint-disable-next-line no-console
          console.error("this is error", err);
        });
    },
  },
};
</script>

<!-- <style scoped>
  .button {
    /* Add your styles here */
  }
  .token {
    /* Add your styles here */
  }
  </style> -->
