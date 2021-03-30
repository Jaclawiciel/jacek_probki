<template>
  <div :class="'module-size-' + size">
    <section class="module module-calendar">
      <button id="hide" v-on:click="$emit('hide-module', 'v-calendar')">
        <img src="../../../../assets/icons/close.svg" alt="close">
      </button>
      <h2 class="module-title">{{moduleTitle}}</h2>
      <div class="module-content">
        <div class="calendar">
          <div class="months">
            <div @click="prevMonth" class="left-arrow"><img src="../../../../assets/icons/left-arrow.svg"
                                                            alt="Poprzedni miesiąc"></div>
            <div class="month-name">{{moment(currentPeriod).format("MMMM YYYY")}}</div>
            <div @click="nextMonth" class="right-arrow"><img src="../../../../assets/icons/right-arrow.svg"
                                                             alt="Następny miesiąc"></div>
          </div>
          <div class="days">
            <span class="day day-name">P</span>
            <span class="day day-name">W</span>
            <span class="day day-name">Ś</span>
            <span class="day day-name">C</span>
            <span class="day day-name">P</span>
            <span class="day day-name weekend">S</span>
            <span class="day day-name weekend">N</span>
            <div class="day-name-border"></div>
            <v-calendar-day v-for="calendarDay in calendarDays" :key="calendarDay.datetime_start"
                            :calendarDay="calendarDay"
                            :prev-month="prevMonth" :next-month="nextMonth"/>
          </div>
        </div>
      </div>
      <img src="../../../../assets/icons/calendar-grey.png" class="module-bg-icon"/>
      <button id="module-info"><img src="../../../../assets/icons/info.svg" alt="Additional info about this module">
      </button>
    </section>
  </div>
</template>

<script>
  import {mapActions, mapGetters} from 'vuex'
  import VNextWorkingDay from "../VNextWorkingDay";
  import VCalendarDay from "./VCalendarDay";
  import VAddModule from "../../VAddModule";
  import SvgIcon from "../../../global/SvgIcon";

  export default {
    name: "VCalendar",
    components: {SvgIcon, VNextWorkingDay, VCalendarDay, VAddModule},
    props: ["currentPeriod", "prevMonth", "nextMonth"],
    data() {
      return {
        // moment: this.moment,
        size: 1,
        moduleTitle: 'Kalendarz',
        // TODO: Fetch scheduled days for currentPeriod
        // schedules: []
      }
    },
    computed: {
      ...mapGetters('workScheduler', ["getWorkScheduler"]),
      ...mapGetters('employers', ["currentEmployer"]),

      calendarDays() {
        var calendarDays = [];
        var currentMonth = this.moment(this.currentPeriod);
        let schedules = this.getWorkScheduler;


        let getSchedulesForDate = (date) => {
          return schedules.filter(schedule => {
            let startDate = this.moment.utc(schedule.datetime_start).format("YYYY-MM-DD");
            let endDate = this.moment.utc(schedule.datetime_stop).format("YYYY-MM-DD");
            return date === startDate || schedule === endDate;
          });
        };

        let getCalendarDay = (date, prevOrNext) => {
          date = this.moment(date);
          date = date.format("YYYY-MM-DD");
          return {
            date: date,
            isPrevMonth: prevOrNext === 'prev',
            isNextMonth: prevOrNext === 'next',
            schedules: getSchedulesForDate(date)
          };
        };

        // Get previous months day in the first week
        currentMonth.date(1);
        while (currentMonth.day() !== 1) {
          currentMonth.subtract(1, "day");
          calendarDays.unshift(getCalendarDay(currentMonth, 'prev'));
        }

        // Get current month days
        currentMonth = this.moment(this.currentPeriod).date(1);
        while (currentMonth.month() === this.moment(this.currentPeriod).month()) {
          calendarDays.push(getCalendarDay(currentMonth));
          currentMonth.add(1, "day");
        }

        // Get next month days
        currentMonth = this.moment(this.currentPeriod);
        currentMonth.endOf("month");
        while (currentMonth.day() !== 0) {
          currentMonth.add(1, "day");
          calendarDays.push(getCalendarDay(currentMonth, 'next'));
        }
        return calendarDays;
      }
    },
    watch: {
      currentEmployer(val) {
        // console.log("Change current employer: ", val);
        this.updateResults();
      }
    },
    mounted: function () {
      this.updateResults();
    },
    methods:
      {
        ...mapActions('workScheduler', ["fetchWorkScheduler"]),
        updateResults() {
          // console.log("Update results, employer: ", this.currentEmployer, ", period:", this.currentPeriod);
          this.fetchWorkScheduler({
            employer: this.currentEmployer.id,
            date_lte: this.moment(this.currentPeriod).add(1, "month").startOf('month').format("YYYY-MM-DD"),
            date_gte: this.moment(this.currentPeriod).startOf('month').format("YYYY-MM-DD"),
            limit: null,
          });
        }
        // updateCalendarDaysWithSchedules() {
        //   let schedules = this.schedules;
        //   this.calendarDays.forEach(function (calendarDay) {
        //     calendarDay.schedules = [];
        //     schedules.forEach(function (schedule) {
        //       if (calendarDay.date === schedule.date) {
        //         calendarDay.schedules.push(schedule);
        //       }
        //     })
        //   })
        // }
      }
  }
</script>

<style scoped>
  .module {
    overflow-x: hidden;
  }

  .module-bg-icon {
    position: absolute;
    top: 100px;
    right: -80px;
    height: 200px;
    opacity: 0.5;
    transform: rotate(-30deg);
  }

  .module-mask {
    clip-path: inset(0);
    height: 100%;
  }

  .calendar {
    margin: 25px 0 50px;
    --calendarWidth: 80vw;
    --gutter: 10px;
    --noOfColumns: 7;
    --rowHeight: calc((var(--calendarWidth) - ((var(--noOfColumns) - 1) * var(--gutter))) / var(--noOfColumns));
    --ratioA: 1;
    --ratioB: 1;
    position: relative;
    z-index: 1;
  }

  .months {
    display: flex;
    justify-content: space-evenly;
    align-items: baseline;
    font-size: 1.125em;
    font-weight: 200;
  }

  .left-arrow img,
  .right-arrow img {
    width: 30px;
    cursor: pointer;
  }

  .calendar .month-name {
    width: 140px;
    text-align: center;
  }

  .calendar .days {
    max-width: var(--calendarWidth);
    margin: 20px auto 0;
    display: grid;
    grid-template-columns: repeat(7, 1fr);
    grid-template-rows: auto 1px repeat(5, var(--rowHeight));
    grid-gap: var(--gutter);
  }

  .calendar .day-name {
    color: var(--darkGreen);
    font-weight: 600;
    font-size: 1.125em;
    cursor: default;
  }

  .day-name-border {
    grid-column-start: 1;
    grid-column-end: 8;
    background: var(--darkGreen);
  }

  .calendar .day {
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .calendar .day.other-month {
    opacity: .2;
  }

  .calendar .day:not(.day-name) {
    border-radius: 50px;
    font-weight: 300;
    cursor: pointer;
  }

  .calendar .weekend {
    opacity: .8;
    color: var(--errorColor);
  }


  /******* MEDIA QUERIES *******/
  /* iPhone SE */
  @media only screen
  and (min-width: 375px) {
  }

  /* iPhone 8 */
  @media only screen
  and (min-width: 375px) {
    .calendar .day {
      font-size: 1.125em;
    }

    .calendar .day-name {
      font-size: 1.25em;
    }
  }

  /* iPhone 8 Plus */
  @media only screen
  and (min-width: 414px) {
    .module-bg-icon {
      top: 120px;
    }

    .calendar .day:not(.day-name) {
      font-size: 1.25em;
    }

    .calendar {
      --gutter: 12px;
    }

    .calendar .days {
      margin: 30px auto 0;
    }
  }

  /* iPhone SE landscape */
  @media only screen
  and (min-width: 568px) {
    .calendar {
      --gutter: 10px;
      --calendarWidth: 65vw;
    }

    .calendar .months {
      justify-content: center;
    }

    .calendar .month-name {
      margin: 0 60px;
      width: 160px;
      font-size: 1.25em;
    }

    .calendar .days {
      margin: 35px auto 0;
    }
  }

  /* iPhone 8 landscape */
  @media only screen
  and (min-width: 667px) {
    .module-bg-icon {
      top: 150px;
    }

    .calendar .month-name {
      font-size: 1.375em;
      width: 200px;
    }
  }

  /* iPhone 8 Plus landscape */
  @media only screen
  and (min-width: 736px) {
    .calendar {
      --calendarWidth: 55vw;
    }
  }

  /* iPhone X landscape */
  @media only screen
  and (min-width: 812px) {
    .calendar {
      --calendarWidth: 50vw;
    }
  }

  /* iPad */
  @media only screen
  and (min-width: 768px) {
    .module-bg-icon {
      top: 210px;
      height: 220px;
    }

    .calendar {
      --gutter: 18px;
      --calendarWidth: 70vw;
    }

    .calendar .month-name {
      font-size: 1.375em;
      width: 180px;
    }

    .calendar .days {
      margin: 45px auto 10px;
    }

    .calendar .day-name {
      font-size: 1.5em;
    }

    .calendar .day:not(.day-name) {
      font-size: 1.5em;
    }
  }

  /* Desktop */
  @media only screen
  and (min-width: 1024px) {
    .module-bg-icon {
      top: 180px;
    }

    .calendar .month-name {
      font-size: 1.75em;
      width: 220px;
    }

    .calendar {
      --calendarWidth: 400px;
    }
  }
</style>
