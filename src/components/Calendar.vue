<template>
  <div>
    <div class="container mt-5">
      <full-calendar :options="calendarOptions"></full-calendar>
    </div>

    <transition name="modal">
      <div class="modal-mask" v-if="showModal">
        <div class="modal-wrapper">
          <div class="modal-container">
            <div class="modal-body">
              <form @submit.prevent="processModalSubmit">
                <div class="input-group mb-3">
                  <label class="input-group-text">Event Title *</label>
                  <input
                    type="text"
                    class="form-control"
                    placeholder="Please add an Event title(min 200 Chars)"
                    v-model.trim="newEvent.eventTitle"
                  />
                </div>
                <div class="input-group mb-3">
                  <label class="input-group-text">Event Description *</label>
                  <textarea
                    type="text"
                    class="form-control"
                    placeholder="Please add an Event Description(min 500 Chars)"
                    v-model.trim="newEvent.eventDesc"
                  />
                </div>
                <div class="row">
                  <div class="col-md-6">
                    <div class="input-group">
                      <label for="start_date" class="input-group-text"
                        >Start Date</label
                      >
                      <input
                        type="date"
                        class="form-control"
                        :readonly="desableStartDate"
                        v-model.trim="newEvent.start"
                      />
                      <input
                        type="time"
                        class="form-control"
                        v-model.trim="newEvent.startTime"
                      />
                    </div>
                  </div>
                  <div class="col-md-6">
                    <div class="input-group">
                      <label for="end_date" class="input-group-text"
                        >End Date</label
                      >
                      <input
                        type="date"
                        class="form-control"
                        v-model.trim="newEvent.endDate"
                      />
                      <input
                        type="time"
                        class="form-control"
                        v-model.trim="newEvent.endTime"
                      />
                    </div>
                  </div>
                </div>
                <div class="col-md-12 mt-3">
                  <button class="btn btn-md btn-success" submit>
                    Save Event
                  </button>
                  <span
                    class="btn btn-md btn-dark ms-2"
                    @click="showModal = false"
                  >
                    Cancel
                  </span>
                  <span class="small float-end"> * must be filled</span>
                </div>
              </form>
            </div>
          </div>
        </div>
      </div>
    </transition>

    <!-- delete Module  -->
    <transition name="modal">
      <div class="modal-mask" v-if="showDeleteModal">
        <div class="modal-wrapper">
          <div class="modal-container">
            <div class="modal-header">
              <h5 class="modal-title">Delete Event</h5>
              <button
                type="button"
                class="btn-close"
                data-bs-dismiss="modal"
                aria-label="Close"
                @click="showDeleteModal = false"
              ></button>
            </div>
            <div class="modal-body">
              <h5>
                Are you sure you want to delete:
                <span style="color: #0a58ca; text-weight: bold">
                  {{ this.event.title }}</span
                >
              </h5>
              <button
                type="button"
                class="btn btn-danger mt-4"
                @click="deleteEvent(event)"
              >
                Delete
              </button>
            </div>
          </div>
        </div>
      </div>
    </transition>

    <div
      class="text-white bg-danger position-fixed bottom-0 end-0 p-3"
      role="alert"
      aria-live="assertive"
      aria-atomic="true"
      v-if="toast"
      style="z-index: 11"
    >
      <div class="d-flex">
        <div class="toast-body">{{ toastMessage }}</div>
        <button
          type="button"
          class="btn-close btn-close-white me-2 m-auto"
          data-bs-dismiss="toast"
          aria-label="Close"
          @click="toast = false"
        ></button>
      </div>
    </div>
  </div>
</template>

<script>
import "@fullcalendar/core/vdom";
import FullCalendar from "@fullcalendar/vue3";
import dayGridPlugin from "@fullcalendar/daygrid";
import timeGridPlugin from "@fullcalendar/timegrid";
import interactionPlugin from "@fullcalendar/interaction";
import eventDate from "../assets/events.json";

export default {
  name: "App",
  components: {
    FullCalendar,
  },

  data() {
    return {
      calendarOptions: {
        editable: true,
        selectable: true,
        selectMirror: true,
        plugins: [dayGridPlugin, timeGridPlugin, interactionPlugin],
        events: null,
        headerToolbar: {
          left: "prev,next today",
          center: "title",
          right: "dayGridMonth,timeGridWeek,timeGridDay",
        },
        initialView: "dayGridMonth",
        dateClick: this.openModal,
        eventClick: this.clickEvent,
      },
      event: "",
      jsonData: eventDate,
      showModal: false,
      showDeleteModal: false,
      desableStartDate: false,
      toast: false,
      toastMessage: "",
      newEvent: {
        eventTitle: null,
        eventDesc: null,
        start: null,
        endDate: null,
        startTime: null,
        endTime: null,
      },
    };
  },

  mounted() {
    this.calendarOptions.events = eventDate.map((item, id) =>
      Object.assign(item, { id })
    );
  },

  watch: {
    toast: function (val) {
      if (val === true) {
        setTimeout(() => (this.toast = false), 3000);
      }
    },
  },

  methods: {
    openModal(payload) {
      this.showModal = true;
      if (payload) {
        this.desableStartDate = true;
        this.newEvent.start = payload.dateStr;
      }
    },

    processModalSubmit(payload) {
      this.addEvent(payload);

      this.newEvent.eventTitle = "";
      this.newEvent.eventDesc = "";
      this.newEvent.start = "";
      this.newEvent.endDate = "";
      this.newEvent.startTime = "";
      this.newEvent.endTime = "";

      this.showModal = false;
    },

    addEvent() {
      const title = this.newEvent.eventTitle;
      const description = this.newEvent.eventDesc;
      const start = this.newEvent.start;
      const end = this.newEvent.endDate;
      const startTime = this.newEvent.startTime;
      const endTime = this.newEvent.endTime;

      var valide = this.validateEventForm(title, description, start, end);
      if (valide === false) {
        return;
      }

      var identifier = this.calendarOptions.events.length;
      var theID = this.checkIfIdExists(identifier);

      this.calendarOptions.events.push({
        id: theID,
        title: title,
        start: start + " " + startTime,
        end: end + " " + endTime,
        description: description,
      });
    },

    validateEventForm(title, description, start, end) {
      if (!title || !description) {
        this.toast = true;
        this.toastMessage = "No title or Description is given";
        return false;
      }

      if (title.length >= 200) {
        this.toast = true;
        this.toastMessage = "The title is too long";
        return false;
      }
      if (description.length >= 500) {
        this.toast = true;
        this.toastMessage = "The description is too long";
        return false;
      }
      if (!start || !end) {
        this.toast = true;
        this.toastMessage = "Please add the time";
        return false;
      }

      if (end < start) {
        this.toast = true;
        this.toastMessage = "End date must be greater then start date";
        return false;
      }
    },

    checkIfIdExists(identifier) {
      this.calendarOptions.events.forEach((element) => {
        if (element.id === identifier) {
          identifier = identifier + 1;
          this.checkIfIdExists(identifier);
        }
      });

      return identifier;
    },

    clickEvent(payload) {
      this.event = payload.event;
      this.showDeleteModal = true;
    },

    deleteEvent(event) {
      if (!event.id) {
        this.toast = true;
        this.toastMessage = "Cannot Delete event, Please contact Administrator";
        this.showDeleteModal = false;
      } else {
        this.showDeleteModal = false;
        return this.calendarOptions.events.splice(event.id, 1);
      }
    },
  },
};
</script>

<style scoped>
.modal-mask {
  position: fixed;
  z-index: 9998;
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
  width: 1000px;
  margin: 0px auto;
  padding: 20px;
  background-color: #fff;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.33);
  transition: all 0.3s ease;
}

.modal-header h3 {
  margin-top: 0;
  color: #42b983;
}

.modal-default-button {
  float: right;
}

.modal-enter {
  opacity: 0;
}

.modal-leave-active {
  opacity: 0;
}

.modal-enter .modal-container,
.modal-leave-active .modal-container {
  -webkit-transform: scale(1.1);
  transform: scale(1.1);
}
</style>
