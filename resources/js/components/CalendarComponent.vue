<template>
  <div class="container">
    <div class="row justify-content-center">
      <div class="col-md-4">
        <form @submit.prevent>
          <div class="form-group">
            <label for="event_name">Event Name</label>
            <input type="text" id="event_name" class="form-control" v-model="newEvent.event_name">
          </div>
          <div class="row">
            <div class="col-md-6">
              <div class="form-group">
                <label for="start_date">Start Date</label>
                <input
                  type="date"
                  id="start_date"
                  class="form-control"
                  v-model="newEvent.start_date"
                >
              </div>
            </div>
            <div class="col-md-6">
              <div class="form-group">
                <label for="end_date">End Date</label>
                <input type="date" id="end_date" class="form-control" v-model="newEvent.end_date">
              </div>
            </div>
            <div class="col-md-6 mb-4" v-if="addingMode">
              <button class="btn btn-sm btn-primary" @click="addNewEvent">Save Event</button>
            </div>
            
          </div>
        </form>
      </div>
      <div class="col-md-8">
        <!-- <Fullcalendar @eventClick="showEvent" :plugins="calendarPlugins" :events="events"/> -->
         <Fullcalendar @eventClick="showEvent" :plugins="calendarPlugins" :events="events"  />
      </div>
    </div>
  </div>
</template>

<script>
import Fullcalendar from "@fullcalendar/vue";
import dayGridPlugin from "@fullcalendar/daygrid";
import interactionPlugin from "@fullcalendar/interaction";
import timeGridPlugin from '@fullcalendar/timegrid';
import listPlugin from '@fullcalendar/list';
import axios from "axios";
export default {
  components: {
    Fullcalendar
  },
  data() {
    return {
      calendarPlugins: [interactionPlugin,timeGridPlugin,listPlugin],
      defaultView: 'listWeek',
      events: "",
      newEvent: {
        event_name: "",
        start_date: "",
        end_date: ""
      },
      addingMode: true,
      indexToUpdate: ""
    };
  },
  created() {
    this.getEvents();
  },
  methods: {
    handleWeekendsToggle() {
      this.calendarPlugins.weekends = !this.calendarPlugins.weekends // update a property
    },
    addNewEvent() {
      axios
        .post("/api/calendar", {
          ...this.newEvent
        })
        .then(data => {
          this.getEvents(); // update our list of events
          this.resetForm();
        })
        .catch(err =>
          console.log("Unable to add new event!", err.response.data)
        );
    },
    showEvent(arg) {
      this.addingMode = false;
      const { id, title, start, end,daysofweek} = this.events.find(
        event => event.id === +arg.event.id
      );
      this.indexToUpdate = id;
      this.newEvent = {
        event_name: title,
        start_date: start,
        end_date: end
      };
    },
    
    getEvents() {
      axios
        .get("/api/calendar")
        .then(resp => (this.events = resp.data.data))
        .catch(err => console.log(err.response.data));
    },
    resetForm() {
      Object.keys(this.newEvent).forEach(key => {
        return (this.newEvent[key] = "");
      });
    }
  },
  watch: {
    indexToUpdate() {
      return this.indexToUpdate;
    }
  }
};
</script>

<style lang="css">
@import "~@fullcalendar/core/main.css";
@import "~@fullcalendar/daygrid/main.css";
.fc-title {
  color: #fff;
}
.fc-title:hover {
  cursor: pointer;
}
</style>
