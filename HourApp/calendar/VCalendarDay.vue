<template>
  <div
    class="day"
    :class="{'today': isToday, 'other-month': isOtherMonth, weekend: isWeekend, 'in-schedule': isInSchedule, 'night-shift-begin': isNightShiftBegin, 'night-shift-end': isNightShiftEnd}"
    :style="[pieChart, nightShiftBoxShadow]"
  >
    <span class="date-number" v-on="smartClick()">
      {{date}}
      <VSchedulesPopover v-if="showPopover" v-model="showPopover" :schedules="calendarDay.schedules" :date="calendarDay.date"
                         @edit-schedule="editSchedule" @add-schedule="addSchedule"/>
    </span>
    <div @click="showPopover = false" :class="{active: showPopover}" class="popover-backdrop"></div>

    <!--todo: inicjalizacja dopiero wtedy, kiedy ktoś kliknie-->
    <v-add-schedule-popup v-if="showEditOrNewPopup" v-model="showEditOrNewPopup" :calendarDay="calendarDay" :scheduleIndex="scheduleIndex"
                          :popupType="popupType"></v-add-schedule-popup>
  </div>
</template>

<script>
  // import moment from 'moment';
  import VSchedulesPopover from "./VSchedulesPopover";
  import VAddSchedulePopup from "./VAddSchedulePopup";
  import {mapGetters} from "vuex";

  export default {
    name: "VCalendarDay",
    components: {VAddSchedulePopup, VSchedulesPopover},
    props: ["calendarDay", "prevMonth", "nextMonth"],
    data() {
      return {
        showPopover: false,
        showEditOrNewPopup: false,
        popupType: null,
        scheduleIndex: 0,
      }
    },
    computed: {
      ...mapGetters('employers', ["currentEmployer"]),

      date() {
        return this.moment(this.calendarDay.date).date()
      },
      isToday() {
        return this.moment(this.calendarDay.date).isSame(this.moment(), 'day');
      },
      isLastDayOfWeek() {
        let day = this.moment(this.calendarDay.date).day();
        return day === 0;
      },
      isWeekend() {
        let day = this.moment(this.calendarDay.date).day();
        return day === 0 || day === 6;
      },
      isOtherMonth() {
        return this.calendarDay.isPrevMonth || this.calendarDay.isNextMonth;
      },
      isPrevMonth() {
        return this.calendarDay.isPrevMonth;
      },
      isNextMonth() {
        return this.calendarDay.isNextMonth;
      },
      isInSchedule() {
        return this.calendarDay.schedules.length > 0;
      },
      isOneSchedule() {
        return this.calendarDay.schedules.length == 1;
      },
      isMultipleSchedule() {
        return this.calendarDay.schedules.length > 1;
      },
      isNightShiftBegin() {
        let currentDate = this.moment.utc(this.calendarDay.date);
        let isNightShiftBegin = false;
        this.calendarDay.schedules.forEach((schedule) => {
          let datetime_stop = this.moment.utc(schedule.datetime_stop);
          if (datetime_stop.isAfter(currentDate, "date")) {
            isNightShiftBegin = true;
          }
        });
        return isNightShiftBegin;
      },
      isNightShiftEnd() {
        let currentDate = this.moment.utc(this.calendarDay.date);
        let isNightShiftEnd = false;
        this.calendarDay.schedules.forEach((schedule) => {
          if (this.moment.utc(schedule.datetime_start).isBefore(currentDate, "date")) {
            isNightShiftEnd = true;
          }
        });
        return isNightShiftEnd;
      },
      isNightShift() {
        return this.isNightShiftBegin || this.isNightShiftEnd;
      },
      nightShiftBoxShadow() {
        if (!this.isNightShiftBegin || this.isLastDayOfWeek) {
          return
        }
        let currentDate = this.moment.utc(this.calendarDay.date);
        let nightShiftEmployerColor;
        this.calendarDay.schedules.forEach((schedule) => {
          if (this.moment.utc(schedule.datetime_stop).isAfter(currentDate, "date")) {
            nightShiftEmployerColor = schedule.employer.color;
          }
        });
        return {'box-shadow': `var(--gutter) 0 0 var(--${nightShiftEmployerColor})`};
      },
      distinctColors() {
        var colors = [];
        this.calendarDay.schedules.forEach((schedule) => {
          if (!colors.contains(schedule.employer.color)) {
            colors.push(schedule.employer.color);
          }
        });
        return colors;
      },
      pieChart() {
        if (!this.isInSchedule) {
          return
        }
        let colors = this.distinctColors.reverse();
        let noOfColors = colors.length;
        let percentage = 100 / noOfColors;
        var style = "conic-gradient(";
        colors.forEach((color, index) => {
          style += ` var(--${color}) ${percentage * index}% ${percentage * (index + 1)}%`;
          if (index !== noOfColors - 1) {
            style += ","
          } else {
            style += ")"
          }
        });

        return {background: style};
      },
    },
    methods: {
      addSchedule(index) {
        this.showEditOrNewPopup = true;
        this.popupType = "add";
        this.scheduleIndex = index;
      },
      editSchedule(data) {
        console.log("Edit multiple schedule: ", data.schedule, data.index);
        this.showEditOrNewPopup = true;
        this.popupType = "edit";
        this.scheduleIndex = data.index
      },
      smartClick() {
        // console.log(this.isInSchedule)
        if (this.isPrevMonth) {
          return {click: this.prevMonth}
        } else if (this.isNextMonth) {
          return {click: this.nextMonth}
        } else if (this.isOneSchedule || this.isMultipleSchedule) {
          return {
            click: () => {
              this.showPopover = true
            }
          }
          // } else if (this.isOneSchedule) {
          //   return {
          //     click: () => {
          //       this.showEditOrNewPopup = true;
          //       this.popupType = "edit";
          //       this.scheduleIndex = 0;
          //     }
          //   }
        } else if (!this.isInSchedule) {
          return {
            click: () => {
              this.showEditOrNewPopup = true;
              this.popupType = "add";
              this.scheduleIndex = 0
            }
          }
        }
      }
    }
  }
</script>

<style scoped>
  .day:hover {
    background: var(--lightGreen) !important;
    color: white !important;
    opacity: 1;
  }

  .day {
    transition: all .15s;
    /*position: relative;*/
  }

  .day.today {
    border: 1px solid var(--darkGreen);
  }

  .day.in-schedule {
    background: var(--lightGreen);
    color: white;
    opacity: 1;
  }

  .day.night-shift-begin {
    border-radius: 50px 0 0 50px !important;
  }

  .day.night-shift-end {
    border-radius: 0 50px 50px 0 !important;
  }

  .popover-backdrop {
    transition: background .5s;
    background: rgba(0, 0, 0, 0);
  }

  .popover-backdrop.active {
    position: fixed;
    width: 100%;
    height: 100%;
    left: 0;
    top: 0;
    z-index: 1;
    cursor: default;
    animation: fadeIn .5s;
    background: rgba(255, 255, 255, 0.5);
  }

  /******* MEDIA QUERIES *******/
  /* iPhone SE */
  @media only screen
  and (min-width: 320px) {
    .date-number {
      padding: 6px;
    }
  }

  /* iPhone SE landscape */
  @media only screen
  and (min-width: 568px) {
    .date-number {
      padding: 10px;
    }
  }

  /* iPhone 8 landscape */
  @media only screen
  and (min-width: 667px) {
    .date-number {
      padding: 14px;
    }
  }

  /* iPad */
  @media only screen
  and (min-width: 768px) {
    .date-number {
      padding: 16px;
    }
  }

  /* Desktop */
  @media only screen
  and (min-width: 1024px) {
    .date-number {
      padding: 6px;
    }
  }
</style>
