<template>
  <div class="container">
    <Transition name="modal">
      <div v-if="show || showNew" class="modal-mask">
        <div class="modal-wrapper">
          <div class="modal-container">
            <div class="modal-header">
              <h3 v-if="show" name="header">Update event</h3>
              <h3 v-if="showNew" name="header">Add event</h3>
            </div>
            <div class="modal-body">
              <form v-if="show" @submit="updateEvent" class="edit">
                <label for="">Title<br /><input type="text" v-model="title" required /><br /></label>
                <label for="">Description<br /><input type="text" v-model="description" required /><br /></label>
                <label for="">Attendees<br /><input type="email" v-model="email" required /><br /></label>
                <label for="">Start<br /><input type="datetime-local" v-model="start" required /><br /></label>
                <label for="">End<br /><input type="datetime-local" :min="start" v-model="end" required /><br /></label>
                <div class="btn">
                  <button type="submit" class="btnSub">Update</button><br />
                  <button type="reset" class="btnDel" @click="handleDelEvent">Delete</button>
                  <button type="reset" class="btnCan" @click="handleCancel">Close</button>
                </div>
              </form>
              <form v-if="showNew" @submit="newEvent" class="new">
                <label for="">Title<br /><input type="text" v-model="title" required /><br /></label>
                <label for="">Description<br /> <input type="text" v-model="description" required /><br /></label>
                <label for="">Attendees<br /><input type="email" v-model="email" required /><br /></label>
                <label for="">Start<br /><input type="datetime-local" v-model="start" required /><br /></label>
                <label for="">End<br /><input type="datetime-local" :min="start" v-model="end" required /><br /></label>
                <div class="btn">
                  <button type="submit" class="btnSub">create</button><br />
                  <button type="reset" class="btnCan" @click="handleCancel">Close</button>
                </div>
              </form>
            </div>
          </div>
        </div>
      </div>
    </Transition>
  </div>
  <FullCalendar :options="calendarOptions" />
</template>
<script>
import moment from "moment"
import FullCalendar from '@fullcalendar/vue3'
import dayGridPlugin from '@fullcalendar/daygrid'
import timeGridPlugin from '@fullcalendar/timegrid'
import listPlugin from '@fullcalendar/list'
import interactionPlugin, { Draggable } from '@fullcalendar/interaction'
import googleCalendarPlugin from '@fullcalendar/google-calendar';

const API_KEY = "AIzaSyAB9RmKSSvZznt81ofOnUG7aXo7qkAUNJk"
export default {
  components: {
    FullCalendar
  },
  props: ['items'],
  data() {
    return {
      calendarOptions: {
        plugins: [googleCalendarPlugin, dayGridPlugin, timeGridPlugin, listPlugin, interactionPlugin],
        initialView: 'timeGridWeek',
        headerToolbar: {
          left: 'prev,next today',
          center: 'title',
          right: 'dayGridMonth,timeGridWeek,timeGridDay,listDay'
        },
        editable: true,
        dayMaxEvents: true, // when too many events in a day, show the popover
        selectable: true,
        droppable: true,
        weekends: true,
        dayMaxEvents: true,
        events: this.items.map(it => {
          return ({
            id: it.id,
            title: it.summary,
            start: moment(it.start.dateTime).format(),
            end: moment(it.end.dateTime).format(),
            "description": it?.description || "NA",
            "summary": it.summary,
            "attendees": it.attendees
          })
        }),
        select: (arg) => {
          this.showNew = true
          this.start = moment(arg.start).format('YYYY-MM-DDTHH:mm:ss')
          this.end = moment(arg.end).format('YYYY-MM-DDTHH:mm:ss')
        },
        eventClick: (arg) => {
          this.selectedEvent = new Object(arg.event)
          this.start = moment(arg.event.start).format('YYYY-MM-DDTHH:mm:ss')
          this.end = moment(arg.event.end).format('YYYY-MM-DDTHH:mm:ss')
          this.title = this.selectedEvent.title
          this.email = this.selectedEvent.extendedProps?.attendees?.[0]?.email
          this.description = this.selectedEvent.extendedProps.description
          this.show = true
        },
        eventDrop: (arg) => {
          this.selectedEvent = new Object(arg.event)
          this.start = moment(arg.event.start).format('YYYY-MM-DDTHH:mm:ss')
          this.end = moment(arg.event.end).format('YYYY-MM-DDTHH:mm:ss')
          this.patchEvent()
        },
        eventResize: (arg) => {
          this.selectedEvent = new Object(arg.event)
          this.start = moment(arg.event.start).format('YYYY-MM-DDTHH:mm:ss')
          this.end = moment(arg.event.end).format('YYYY-MM-DDTHH:mm:ss')
          this.patchEvent()
        }
      },
      selectedEvent: "",
      show: false,
      title: "",
      showNew: false,
      start: "",
      end: "",
      description: "",
      email: "",
      id: "",
    }
  },

  methods: {
    handleSubmit(event) {
      event.preventDefault()
      this.calendarOptions.events.find((it) => {
        if (it.id == this.selectedEvent.id) {
          it.title = this.title,
            it.start = moment(this.start).format()
          it.end = moment(this.end).format()
        }
      })
      this.show = false;
      this.title = ""
      this.start = ""
      this.end = ""
      this.email = ""
    },
    newEvent(event) {
      event.preventDefault()
      let data = {
        "end": { "dateTime": moment(this.end).format(), "timeZone": "UTC" },
        "start": { "dateTime": moment(this.start).format(), "timeZone": "UTC" },
        "description": this.description,
        "summary": this.title,
        'attendees': [
          { 'email': this.email }
        ],
        reminders: {
          'useDefault': false,
          'overrides': [
            { 'method': 'email', 'minutes': 24 * 60 },
            { 'method': 'popup', 'minutes': 10 }
          ]
        }
      }
      this.createEvent(data)
    },

    createEvent(eventData) {
      let token = JSON.parse(localStorage.getItem('oauth2-test-params'))
      if (token) {
        const requestOptions = {
          method: "POST",
          headers: {
            "content-type": "application/json",
            authorization: `Bearer ${token.access_token}`,
          },
          body: JSON.stringify(eventData)
        };
        fetch(`https://www.googleapis.com/calendar/v3/calendars/primary/events?key=${API_KEY}`, requestOptions)
          .then((response) => response.json()
          )
          .then(() => {
            this.$parent.list()
            window.alert("Successfully created event")
            this.handleCancel()
          })
          .catch((error) => console.log(error))
      }
      else {
        this.$parent.oauthSignIn()
      }
    },
    handleDelEvent() {
      let token = JSON.parse(localStorage.getItem('oauth2-test-params'))
      if (token) {
        const requestOptions = {
          method: "DELETE",
          headers: {
            "content-type": "application/json",
            authorization: `Bearer ${token.access_token}`,
          },
        };
        fetch(`https://www.googleapis.com/calendar/v3/calendars/primary/events/${this.selectedEvent.id}?key=${API_KEY}`, requestOptions)
          .then((response) => {
            if (response.ok) {
              window.alert("deleted successfully")
              this.$parent.list()
              this.handleCancel()
            }
            else {
              window.alert("Please try again")
            }
          }
          )
          .catch((error) => console.log(error))
      }
    },
    updateEvent(event) {
      event.preventDefault()
      let data = {
        id: this.selectedEvent.id,
        end: { "dateTime": moment(this.end).format(), "timeZone": "UTC" },
        start: { "dateTime": moment(this.start).format(), "timeZone": "UTC" },
        "description": this.description,
        "summary": this.title,
        'attendees': [
          { 'email': this.email }
        ],
        reminders: {
          'useDefault': false,
          'overrides': [
            { 'method': 'email', 'minutes': 24 * 60 },
            { 'method': 'popup', 'minutes': 10 }
          ]
        }
      }
      let token = JSON.parse(localStorage.getItem('oauth2-test-params'))
      if (token) {
        const requestOptions = {
          method: "PUT",
          headers: {
            "content-type": "application/json",
            authorization: `Bearer ${token.access_token}`,
          },
          body: JSON.stringify(data)
        };
        fetch(`https://www.googleapis.com/calendar/v3/calendars/primary/events/${this.selectedEvent.id}?key=${API_KEY}`, requestOptions)
          .then((response) => {
            if (response.ok) {
              window.alert("updated successfully")
              this.$parent.list()
              this.handleCancel()
            }
            else {
              window.alert("Please try again")
            }
          }
          )
          .catch((error) => console.log(error))
      }
    },
    patchEvent() {
      let data = {
        id: this.selectedEvent.id,
        end: { "dateTime": moment(this.selectedEvent.end).format(), "timeZone": "UTC" },
        start: { "dateTime": moment(this.selectedEvent.start).format(), "timeZone": "UTC" },
      }
      let token = JSON.parse(localStorage.getItem('oauth2-test-params'))
      if (token) {
        const requestOptions = {
          method: "PATCH",
          headers: {
            "content-type": "application/json",
            authorization: `Bearer ${token.access_token}`,
          },
          body: JSON.stringify(data)
        };
        fetch(`https://www.googleapis.com/calendar/v3/calendars/primary/events/${this.selectedEvent.id}?key=${API_KEY}`, requestOptions)
          .then((response) => {
            if (response.ok) {
              window.alert("updated successfully")
              this.$parent.list()
              this.handleCancel()
            }
            else {
              window.alert("Please try again")
              location.reload()
            }
          }
          )
          .catch((error) => console.log(error))
      }
    },
    handleCancel() {
      this.showNew = false
      this.selectedEvent = ""
      this.show = false
      this.title = ""
      this.start = ""
      this.end = ""
      this.description = ""
      this.email = ""
      this.id = ""
    },
  }
}
</script>
<style >
.container {
  display: block;
}

.calender {
  padding: 25px;
  margin: 20px;
}

.edit,
.new {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px
}

.modal-mask {
  position: fixed;
  z-index: 10000;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: table;
  transition: opacity 0.3s ease;
}

.modal-wrapper {
  display: table-cell;
  vertical-align: middle;
}

.modal-container {
  width: 300px;
  margin: 0px auto;
  padding: 20px 30px;
  background-color: #fff;
  border-radius: 2px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.33);
  transition: all 0.3s ease;
}

.modal-header h3 {
  margin-top: 0;
  color: #42b983;
}

.modal-body {
  margin: 20px 0;
}

.btn {
  display: flex;
}

.btnSub {
  background-color: #42b983;
  color: #fff;
  margin: 5px;
  padding: 10px 15px;
  cursor: pointer;
  border: none;
  border-radius: 10px;
}

.btnDel {
  background-color: gray;
  color: #fff;
  margin: 5px;
  padding: 10px 15px;
  cursor: pointer;
  border: none;
  border-radius: 10px;
}

.btnCan {
  background-color: tomato;
  color: #fff;
  margin: 5px;
  padding: 10px 15px;
  cursor: pointer;
  border: none;
  border-radius: 10px;
}

.btnSub:hover,
.btnCan:hover,
.btnDel:hover {
  opacity: 75%;
  transform: scale(1.1);
}
</style>
