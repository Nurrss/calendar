<template>
  <button @click="authenticate().then(loadClient)">authorize and load</button>
  <button @click="execute()">execute</button>
</template>
<script>
export default {
  data() {
    return {
      eventTitle: "",
      eventStart: "",
      eventEnd: "",
    };
  },
  mounted() {
    gapi.load("client:auth2", function () {
      gapi.auth2.init({
        client_id:
          "187108771123-pgv3okbdldvhv1d25htf20t0j7undbu3.apps.googleusercontent.com",
      });
    });
  },
  methods: {
    authenticate() {
      return gapi.auth2
        .getAuthInstance()
        .signIn({
          scope:
            "https://www.googleapis.com/auth/calendar https://www.googleapis.com/auth/calendar.events https://www.googleapis.com/auth/calendar.events.readonly https://www.googleapis.com/auth/calendar.readonly",
        })
        .then(
          function () {
            console.log("Sign-in successful");
          },
          function (err) {
            console.error("Error signing in", err);
          }
        );
    },
    loadClient() {
      gapi.client.setApiKey(
        "187108771123-pgv3okbdldvhv1d25htf20t0j7undbu3.apps.googleusercontent.com"
      );
      return gapi.client
        .load(
          "https://content.googleapis.com/discovery/v1/apis/calendar/v3/rest"
        )
        .then(
          function () {
            console.log("GAPI client loaded for API");
          },
          function (err) {
            console.error("Error loading GAPI client for API", err);
          }
        );
    },
    execute() {
      return gapi.client.calendar.events
        .list({
          calendarId: "primary",
        })
        .then(
          function (response) {
            // Handle the results here (response.result has the parsed body).
            console.log("Response", response);
          },
          function (err) {
            console.error("Execute error", err);
          }
        );
    },
  },
};
</script>
