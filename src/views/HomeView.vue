<template>
  <button type='button' class="btn-login" v-if="!authorized" @click='oauthSignIn'>Login</button>
  <button type='button' class="btn-logout" v-if="authorized" @click='logout'>Logout</button>
  <div class="calender" v-if="authorized">
    <Calender :items="calenderData" />
  </div>
</template>
<script>
import Calender from "../components/Calender.vue"

const API_KEY = import.meta.env.VITE_API_KEY;

const CLIENT_ID = import.meta.env.VITE_CLIENT_ID;
const SCOPES = "https://www.googleapis.com/auth/calendar https://www.googleapis.com/auth/calendar.events https://www.googleapis.com/auth/calendar.events.readonly https://www.googleapis.com/auth/calendar.readonly"
export default {
  components: {
    Calender
  },
  data() {
    return {
      authorized: false,
      calenderData: [],
      selected: {},
      show: false,
    }
  },
  mounted() {
    if (this.$route.hash) {
      this.getToken();
    }
    let token = JSON.parse(localStorage.getItem('oauth2-test-params'))
    if (token) {
      this.authorized = true
      this.list()
    }
  },
  methods: {
    oauthSignIn() {
      // Google's OAuth 2.0 endpoint for requesting an access token
      var oauth2Endpoint = 'https://accounts.google.com/o/oauth2/v2/auth';

      // Create <form> element to submit parameters to OAuth 2.0 endpoint.
      var form = document.createElement('form');
      form.setAttribute('method', 'GET'); // Send as a GET request.
      form.setAttribute('action', oauth2Endpoint);

      // Parameters to pass to OAuth 2.0 endpoint.
      var params = {
        'client_id': CLIENT_ID,
        'redirect_uri': 'http://localhost:5173',
        'response_type': 'token',
        'scope': SCOPES,
        'include_granted_scopes': 'true',
        'state': 'pass-through value'
      };

      // Add form parameters as hidden input values.
      for (var p in params) {
        var input = document.createElement('input');
        input.setAttribute('type', 'hidden');
        input.setAttribute('name', p);
        input.setAttribute('value', params[p]);
        form.appendChild(input);
      }

      // Add form to page and submit it to open the OAuth 2.0 endpoint.
      document.body.appendChild(form);
      form.submit();
      this.getToken();
    },
    logout() {
      localStorage.clear()
      this.authorized = false
    },
    getToken() {
      let params = {}
      var regex = /([^&=]+)=([^&]*)/g, m;
      if (this.$route.hash) {
        while (m = regex.exec(this.$route.hash)) {
          params[decodeURIComponent(m[1])] = decodeURIComponent(m[2]);
        }
        if (Object.keys(params).length > 0) {
          localStorage.setItem('oauth2-test-params', JSON.stringify(params));
          if (params['state'] && params['state'] == 'try_sample_request') {
            trySampleRequest();
          }
        }
        this.$router.replace({ path: '/' })
        this.list()
        this.authorized = true
      }
    },
    list() {
      let token = JSON.parse(localStorage.getItem('oauth2-test-params'))
      this.authorized = false
      if (token?.access_token) {
        const requestOptions = {
          method: "GET",
          headers: {
            "content-type": "application/json",
            authorization: `Bearer ${token.access_token}`,
          },
        };
        fetch(`https://content.googleapis.com/calendar/v3/calendars/primary/events?key=${API_KEY}`, requestOptions)
          .then((response) => {
            if (response.ok) {
              return response.json()
            }
            else {
              localStorage.clear()
              this.oauthSignIn()
              return
            }
          })
          .then((data) => {
            this.calenderData = data?.items
            this.authorized = true
          }).catch((error) => {
            console.log(error);
            this.authorized = false
          })
        this.$router.replace({ path: '/' })
      }
      else {
        this.oauthSignIn()
      }
    },
  },
}
</script>
<style>
.btn-login {
  background-color: #238636;
  padding: 8px 15px;
  font-weight: 700;
  border: none;
  border-radius: 5px;
  color: white;
  cursor: pointer;
}

.btn-logout {
  background-color: rgb(248, 88, 60);
  padding: 8px 15px;
  font-weight: 700;
  border: none;
  border-radius: 5px;
  color: white;
  cursor: pointer;
}

.btn-login:hover,
.btn-logout:hover {
  opacity: 75%;
  transform: scaleY(110%);
}
</style>
