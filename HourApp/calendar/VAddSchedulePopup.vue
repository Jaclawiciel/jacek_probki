<template>
  <v-popup v-model="value">
    <span slot="header">
        <!--<span v-if="showAddPopup">Nowy wpis w grafiku</span>-->
      <!--<span v-if="showEditPopup">Edycja wpisu w grafiku</span>-->
        <span v-if="isEdit">Edycja wpisu w grafiku</span>
        <span v-else>Nowy wpis w grafiku</span>
    </span>
    <!--</h2>-->
    <div slot="body" v-if="isEdit || isAdd">
      <form action="" id="add-payment">
        <div class="input-container">
          <span class="input-details">Pracodawca</span>
          <select class="filter-text" v-model="currentSchedule.employer.id" @change="changeEmployer" :disabled="isEdit">
            <option disabled="disabled" selected="selected">Wybierz pracodawcę</option>
            <option v-for="employer in employers" :key="employer.id" v-if="employer.id !== -1" :value="employer.id">
              {{employer.name}}
            </option>
          </select>
        </div>
        <div class="input-container">
          <span class="input-details">Data rozpoczęcia</span>
          <flat-pickr v-model="currentSchedule.datetime_start" :config="datetime_config"></flat-pickr>
          <!--<input type="text" name="date" id="date" v-model="pay.datetime">-->
          <!--<span class="input-error-msg hidden">{{error}}</span>-->
        </div>
        <div class="input-container">
          <span class="input-details">Data zakończenia</span>
          <flat-pickr v-model="currentSchedule.datetime_stop" :config="datetime_config"></flat-pickr>
          <!--<input type="text" name="date" id="date" v-model="pay.datetime">-->
          <!--<span class="input-error-msg hidden">{{error}}</span>-->
        </div>
        <div class="input-container">
          <span class="input-details">Stawka za godzinę</span>
          <input type="number" name="value" id="value" required v-model.number="currentSchedule.pay_for_an_hour">
          <!--<span class="input-error-msg hidden">{{error}}</span>-->
        </div>
        <div class="info-msg" v-if="helpTip">
          <img src="../../../../assets/icons/info-green.svg" alt="Informacja o zarobku">
          <span>Zarobisz {{helpTipPayAnimated}} PLN w {{helpTipHoursAnimated}} {{helpTip.hoursText}}</span>
        </div>
      </form>
    </div>
    <div slot="footer">
      <div class="submit-btns">
        <div class="submit-btn">
          <button type="button" class="btn" @click="addSchedule" v-if="isAdd">Zapisz</button>
          <button type="button" class="btn" @click="editSchedule" v-if="isEdit">Zaktualizuj</button>
        </div>
        <div class="submit-btn">
          <button type="button" class="btn secondary" @click="closePopup">Anuluj</button>
        </div>
        <div class="submit-btn delete" v-if="isEdit">
          <button type="button" class="btn danger" @click="deleteSchedule">Usuń wpis z grafiku</button>
        </div>
      </div>
    </div>

  </v-popup>
</template>

<script>
  import VPopup from "../../other/VPopup";
  import flatPickr from 'vue-flatpickr-component';
  import {Polish} from "flatpickr/dist/l10n/pl.js"
  import axios from "axios";
  import {mapActions, mapGetters} from "vuex";
  import {TweenLite} from "gsap/TweenLite";

  export default {
    components: {VPopup, flatPickr},
    props: ["value", "calendarDay", "popupType", "scheduleIndex"],
    data() {
      return {
        datetime_config: {
          disableMobile: false,
          locale: Polish,
          enableTime: true,
          time_24hr: true,
          altFormat:
            "d-m-Y H:i ",
          altInput:
            true,
          dateFormat:
            'Y-m-d\\TH:i',
        },
        scheduleBackup: {},
        currentSchedule: {
          employer: {
            id: ""
          },
          datetime_start: this.moment.utc(this.calendarDay.date + " 08:00").format("YYYY-MM-DD HH:mm"),
          datetime_stop: this.moment.utc(this.calendarDay.date + " 16:00").format("YYYY-MM-DD HH:mm"),
          pay_for_an_hour: ""
        },
        helpTipPayTemp: 0,
        helpTipHoursTemp: 8,
      }
    },
    mounted() {
      if (this.isSchedule()) {
        this.scheduleBackup = this.calendarDay.schedules[this.scheduleIndex];
        this.currentSchedule = Object.assign({}, this.scheduleBackup);
      } else {
        if (this.currentEmployer.id !== -1) {
          this.currentSchedule.pay_for_an_hour = this.currentEmployer.default_pay_for_an_hour;
          this.currentSchedule.employer.id = this.currentEmployer.id;
        }
      }
    },
    computed: {
      ...mapGetters('mainStore', ["getApiURL", "getUserToken"]),
      ...mapGetters('employers', ["employers", "employerById", "currentEmployer"]),
      isEdit() {
        // console.log(this.popupType)
        return this.popupType === "edit";
      },
      isAdd() {
        return this.popupType === "add";
      },
      helpTip() {
        // console.log("Zmiana w currentSchedule")
        let datetime_start = this.moment(this.currentSchedule.datetime_start);
        let datetime_stop = this.moment(this.currentSchedule.datetime_stop);
        let duration = this.moment.duration(datetime_stop.diff(datetime_start));
        let hoursText;
        var hours = duration.asHours();
        var pay = hours * this.currentSchedule.pay_for_an_hour;
        if (hours > 0 && !isNaN(pay)) {
          switch (true) {
            case (hours >= 5):
              hoursText = "godzin";
              break;
            case (hours >= 2):
              hoursText = "godziny";
              break;
            default:
              hoursText = "godzinę";
              break;
          }
          return {hours, pay, hoursText}
        } else {
          return undefined
        }
      },
      helpTipPayAnimated: function () {
        return this.helpTipPayTemp.toFixed(0);
      },
      helpTipHoursAnimated: function () {
        return this.helpTipHoursTemp.toFixed(0);
      }
    },
    watch: {
      helpTip: function (newValue) {
        TweenLite.to(this.$data, 1, {helpTipPayTemp: newValue.pay});
        TweenLite.to(this.$data, 1, {helpTipHoursTemp: newValue.hours});
      }
    },
    methods: {
      ...mapActions('workScheduler', ["fetchWorkScheduler"]),
      closePopup() {
        this.currentSchedule = this.scheduleBackup;
        this.$emit('input', false);
      },
      isSchedule() {
        return (this.calendarDay.schedules[this.scheduleIndex]) ? true : false;
      },

      changeEmployer() {
        let employerData = this.getEmployerById(this.currentSchedule.employer.id);
        // console.log("Zmiana firmy: ", employerData, "PFAN:", employerData.default_pay_for_an_hour);
        this.currentSchedule.pay_for_an_hour = employerData.default_pay_for_an_hour
      },
      addSchedule() {
        // console.log("Add", this.currentSchedule);
        axios({
          method: "post",
          url: `${this.getApiURL}/work_times/`,
          data: {
            ...this.currentSchedule
          },
          headers: {
            'Authorization': 'Token ' + this.getUserToken
          }
        }).then((response) => {
          // console.log("Success");
          this.fetchWorkScheduler({
            employer: this.currentEmployer.id,
            period: this.calendarDay.date,
          });
          // this.fetchWorkScheduler(this.calendarDay.date, this.currentEmployer)
          this.closePopup();
        }).catch((res) => {
          console.log("Error: " + res)
        })
      },
      editSchedule() {
        // console.log("Edit", this.currentSchedule);
        axios({
          method: "patch",
          url: `${this.getApiURL}/work_times/${this.currentSchedule.id}/`,
          data: {
            ...this.currentSchedule
          },
          headers: {
            'Authorization': 'Token ' + this.getUserToken
          }
        }).then((response) => {
          // console.log("Success");
          this.fetchWorkScheduler({
            employer: this.currentEmployer.id,
            period: this.calendarDay.date,
          });
          this.closePopup();
        }).catch((res) => {
          console.log("Error: " + res)
        })
      },
      deleteSchedule() {
        // console.log("Delete", this.currentSchedule);
        axios({
          method: "delete",
          url: `${this.getApiURL}/work_times/${this.currentSchedule.id}/`,
          headers: {
            'Authorization': 'Token ' + this.getUserToken
          }
        }).then((response) => {
          // console.log("Success");
          this.fetchWorkScheduler({
            employer: this.currentEmployer.id,
            period: this.calendarDay.date,
          });
          this.closePopup();
        }).catch((res) => {
          console.log("Error: " + res)
        })
      }
    },
  }
</script>

<style scoped>
  .info-msg {
    display: flex;
    align-items: center;
  }

  .info-msg img {
    margin-right: 8px;
  }

  .submit-btns {
    display: flex;
    justify-content: space-around;
    flex-wrap: wrap;
    margin: 15px 0 25px;
    width: 100%;
  }

  .submit-btns .submit-btn {
    text-align: center;
    width: 50%;
  }

  .submit-btns .submit-btn.delete {
    width: 100%;
    flex-grow: 2;
    margin-bottom: 3px;
  }

  /******* MEDIA QUERIES *******/
  /* iPhone 8 */
  @media only screen
  and (min-width: 375px) {
    form {
      width: 100%;
    }

    .submit-btns .btn {
      padding: 10px 30px;
    }

    .submit-btns .btn:enabled:hover {
      padding: 10px 20px;
      border-radius: 15px;
    }
  }

  /******* MEDIA QUERIES *******/
  /* iPhone SE Landscape */
  @media only screen
  and (min-width: 568px) {
    .wrapper {
      max-width: 400px;
      margin-top: 30px;
    }

    form {
      width: 100%;
    }

    .submit-btns .btn {
      padding: 10px 30px;
    }

    .submit-btns .btn:enabled:hover {
      padding: 10px 20px;
      border-radius: 15px;
    }
  }
</style>
