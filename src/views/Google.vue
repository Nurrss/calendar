<template>
  <div>
    <h2>Create Event</h2>
    <form @submit.prevent="createEvent">
      <label for="title">Title:</label>
      <input id="title" type="text" v-model="eventTitle" />
      <br />
      <label for="start">Start:</label>
      <input id="start" type="datetime-local" v-model="eventStart" />
      <br />
      <label for="end">End:</label>
      <input id="end" type="datetime-local" v-model="eventEnd" />
      <br />
      <button type="submit">Create Event</button>
    </form>
  </div>
</template>

<script>
//   import  gapi  from 'googleapis';

export default {
  data() {
    return {
      eventTitle: "",
      eventStart: "",
      eventEnd: "",
    };
  },
  methods: {
    async createEvent() {
      gapi.load("client:auth2", () => {
        console.log("gapi loaded", gapi.client);
        gapi.client
          .init({
            apiKey: "AIzaSyDpsIZqfO1u8LhY26tZYhZ16SXhxnGXsjw",
            clientId:
              "187108771123-pgv3okbdldvhv1d25htf20t0j7undbu3.apps.googleusercontent.com",
          })
          .then(() => {
            console.log(" gapi.client.init completed", gapi.client);
            gapi.client
              .request({
                path: `https://www.googleapis.com/calendar/v3/calendars/primary/events`,
              })
              .then((response) => {
                //   let events = response.result.items
                console.log(response);
              });
            const gcEvent = {
              summary: "Google I/O 2015",
              location: "800 Howard St., San Francisco, CA 94103",
              description:
                "A chance to hear more about Google's developer products.",
              start: {
                dateTime: "2023-04-01T09:00:00-07:00",
                timeZone: "America/Los_Angeles",
              },
              end: {
                dateTime: "2023-04-01T17:00:00-07:00",
                timeZone: "America/Los_Angeles",
              },
              recurrence: ["RRULE:FREQ=DAILY;COUNT=2"],
              reminders: {
                useDefault: false,
                overrides: [
                  { method: "email", minutes: 24 * 60 },
                  { method: "popup", minutes: 10 },
                ],
              },
            };

            var request = gapi.client.calendar.events.insert({
              calendarId: "primary",
              resource: gcEvent,
            });

            // window object is undefined inside `execute`/
            const rootWindow = window;

            request.execute((gcEvent) => {
              rootWindow.open(gcEvent.htmlLink);
            });
          });
      });
    },
  },
};
</script>
