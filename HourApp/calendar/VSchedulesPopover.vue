<template>
  <div>
    <transition name="slideUp">
      <div class="schedules-popover" v-show="value">
        <div class="date">
          {{moment(date).format("DD MMMM YYYY")}}
        </div>
        <div class="schedules">
          <div class="schedule" v-for="(schedule, index) in schedules">
            <SvgIcon class="employer-icon" :id="schedule.employer.icon" :color="schedule.employer.color" :fill="'fill'"/>
            <div class="times">
              <div class="start-time">
                <SvgIcon class="time-icon" :id="'arrive'" :color="schedule.employer.color" :stroke="'stroke'"/>
                <span>{{moment(schedule.datetime_start).utc().format("HH:mm")}}</span>
                <span v-show="isNightShift(schedule)" class="detailed-date">{{moment(schedule.datetime_start).utc().format("DD MMM")}}</span>
              </div>
              <div class="duration">
                <span>{{workDuration(schedule)}} godz.</span>
                <div class="duration-line" :style="`border-top: 1px dotted var(--${schedule.employer.color});`"></div>
              </div>
              <div class="end-time">
                <SvgIcon class="time-icon" :id="'exit'" :color="schedule.employer.color" :stroke="'stroke'"/>
                <span>{{moment(schedule.datetime_stop).utc().format("HH:mm")}}</span>
                <span v-show="isNightShift(schedule)" class="detailed-date">{{moment(schedule.datetime_stop).utc().format("DD MMM")}}</span>
              </div>
            </div>
            <img class="edit" src="../../../../assets/icons/edit.svg" alt="Edytuj" @click="editSchedule(schedule, index)">
          </div>
        </div>
        <button @click="addSchedule(lastIndex)" class="add-schedule btn secondary action">
          <img src="../../../../assets/icons/add.svg" alt="Dodaj">
          <span>Dodaj</span>
        </button>
      </div>
    </transition>
    <slot name="day"></slot>
  </div>
</template>

<script>
  import moment from 'moment';
  import SvgIcon from "../../../global/SvgIcon";

  export default {
    name: "VSchedulesPopover",
    components: {SvgIcon},
    props: ["showPopover", "schedules", "date", "value"],
    computed: {
      lastIndex() {
        return this.schedules.length;
      }
    },
    methods: {
      isNightShift: function (schedule) {
        return moment.utc(schedule.datetime_start).isBefore(moment.utc(schedule.datetime_stop), "date");
      },
      workDuration: function (schedule) {
        return moment.utc(schedule.datetime_stop).diff(moment.utc(schedule.datetime_start), "hours");
      },
      addSchedule(index) {
        this.$emit('add-schedule', index)
      },
      editSchedule(schedule, index) {
        this.$emit('edit-schedule', {schedule, index})
      }
    }
  }
</script>

<style scoped>
  .schedules-popover {
    position: absolute;
    width: 250px;
    top: -40px;
    left: 0;
    right: 0;
    margin: 0 auto;
    background: white;
    color: var(--fontColor);
    z-index: 2;
    box-shadow: 0 14px 28px rgba(0, 0, 0, 0.25), 0 10px 10px rgba(0, 0, 0, 0.22);
    padding: 10px 0;

    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    cursor: default;
  }

  .schedules-popover .date {
    font-size: 22px;
    font-weight: 400;
    text-align: center;
    margin-bottom: 10px;
  }

  .schedules-popover .schedules {
    width: 100%;
  }

  .schedules-popover .schedule {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 10px 16px;
    background: transparent;
    transition: all .15s;
  }

  .schedules-popover .schedule:hover {
    background: #eee;
  }

  .schedules-popover .schedule .employer-icon {
    width: 30px;
    height: 30px;
  }

  .schedules-popover .schedule .times {
    display: flex;
    justify-content: center;
    align-items: flex-start;
    align-self: center;
  }

  .schedules-popover .schedule .start-time,
  .schedules-popover .schedule .end-time {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    align-self: center;
  }

  .schedules-popover .schedule .time-icon {
    width: 30px;
    height: 20px;
  }

  .schedules-popover .schedule .start-time {
    margin-right: -8px;
  }

  .schedules-popover .schedule .end-time {
    margin-left: -8px;
  }

  .detailed-date {
    font-size: 12px;
    font-weight: 100;
  }

  .schedules-popover .duration {
    font-size: 10px;
    font-weight: 200;
    width: 50px;
    margin-top: -5px;
    text-align: center;
  }

  .schedules-popover .duration-line {
    border-top: 1px dotted var(--orange);
  }

  .add-schedule {
    margin-top: 15px;
  }

  .edit {
    cursor: pointer;
  }

  /* iPhone 8 landscape */
  @media only screen
  and (min-width: 667px) {
    .schedules-popover {
      width: 275px;
      top: 60px;
    }

    .schedules-popover .schedule .time-icon {
      width: 35px;
      height: 25px;
    }

    .schedules-popover .duration {
      font-size: 14px;
      margin-top: -8px;
    }
  }

  /* Desktop HD */
  @media only screen
  and (min-width: 1440px) {
    .edit {
      opacity: 0;
      transition: opacity .15s;
    }

    .schedule:hover .edit {
      opacity: 1;
    }
  }
</style>
