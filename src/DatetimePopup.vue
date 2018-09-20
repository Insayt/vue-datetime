<template>
  <div class="vdatetime-popup" :class="{ inline: inline }">
    <div class="vdatetime-popup__header" v-if="!inline">
      <div class="vdatetime-popup__year" @click="showYear">{{ year }}</div>
      <div class="vdatetime-popup__date">{{ dateFormatted }}</div>
    </div>
    <div class="vdatetime-popup__body">
      <datetime-year-picker
          v-if="step === 'year'"
          @change="onChangeYear"
          :year="year"></datetime-year-picker>
      <datetime-calendar
          @change="onChangeDate"
          :year="year"
          :month="month"
          :day="day"
          :min-date="minDatetime"
          :max-date="maxDatetime"
          :week-start="weekStart"
      ></datetime-calendar>
      <datetime-time-picker
          v-if="step === 'time'"
          @change="onChangeTime"
          @cancelTime="cancelTime"
          :inline="inline"
          :hour="hour"
          :phrases="phrases"
          :minute="minute"
          :use12-hour="use12Hour"
          :hour-step="hourStep"
          :minute-step="minuteStep"
          :min-time="minTime"
          :max-time="maxTime"></datetime-time-picker>
    </div>
    <div class="vdatetime-popup__actions" v-if="!inline">
      <div class="vdatetime-popup__actions__button vdatetime-popup__actions__button--cancel" @click="cancel">{{ phrases.cancel }}</div>
      <div class="vdatetime-popup__actions__button vdatetime-popup__actions__button--confirm" @click="confirm">{{ phrases.ok }}</div>
    </div>
    <div class="vdatetime-popup__inline-bottom">
      <input class="vdatetime-popup__inline-bottom-date" type="text" disabled="disabled" v-model="showDate">
      <input class="vdatetime-popup__inline-bottom-time" type="text" v-model="showTime" @click="showTimePopup">
    </div>
    <div class="vdatetime-popup__inline-bottom _center">
      <select class="vdatetime-popup__inline-bottom-timezone" v-model="timezone" @change="changeTimezone">
        <option value="Europe/Kaliningrad">Калининград (MSK–1)</option>
        <option value="Europe/Moscow">Москва (MSK)</option>
        <option value="Europe/Samara">Самара (MSK+1)</option>
        <option value="Asia/Yekaterinburg">Екатеринбург (MSK+2)</option>
        <option value="Asia/Omsk">Омск (MSK+3)</option>
        <option value="Asia/Krasnoyarsk">Красноярск (MSK+4)</option>
        <option value="Asia/Irkutsk">Иркутск (MSK+5)</option>
        <option value="Asia/Yakutsk">Якутск (MSK+6)</option>
        <option value="Asia/Vladivostok">Владивосток (MSK+7)</option>
        <option value="Asia/Magadan">Магадан (MSK+8)</option>
        <option value="Asia/Kamchatka">Камчатка (MSK+9)</option>
      </select>
    </div>

  </div>
</template>

<script>
import { DateTime } from 'luxon'
import { createFlowManagerFromType } from './util'
import DatetimeCalendar from './DatetimeCalendar'
import DatetimeTimePicker from './DatetimeTimePicker'
import DatetimeYearPicker from './DatetimeYearPicker'

const KEY_TAB = 9
const KEY_ENTER = 13
const KEY_ESC = 27

export default {
  components: {
    DatetimeCalendar,
    DatetimeTimePicker,
    DatetimeYearPicker
  },

  props: {
    datetime: {
      type: DateTime,
      required: true
    },
    phrases: {
      type: Object,
      default () {
        return {
          cancel: 'Cancel',
          ok: 'Ok'
        }
      }
    },
    type: {
      type: String,
      default: 'date'
    },
    use12Hour: {
      type: Boolean,
      default: false
    },
    hourStep: {
      type: Number,
      default: 1
    },
    minuteStep: {
      type: Number,
      default: 1
    },
    minDatetime: {
      type: DateTime,
      default: null
    },
    maxDatetime: {
      type: DateTime,
      default: null
    },
    auto: {
      type: Boolean,
      default: false
    },
    weekStart: {
      type: Number,
      default: 1
    },
    inline: {
      type: Boolean,
      default: false
    },
    zone: {
      type: String,
      default: 'Europe/Moscow'
    }
  },

  data () {
    const flow = createFlowManagerFromType(this.type)

    return {
      newDatetime: this.datetime,
      showDate: this.datetime.toFormat('dd.MM.yyyy'),
      showTime: this.datetime.toFormat('HH:mm'),
      flow: flow,
      step: flow.first(),
      timePartsTouched: [],
      timezone: this.zone
    }
  },

  created () {
    document.addEventListener('keydown', this.onKeyDown)
  },

  beforeDestroy () {
    document.removeEventListener('keydown', this.onKeyDown)
  },

  computed: {
    year () {
      return this.newDatetime.year
    },
    month () {
      return this.newDatetime.month
    },
    day () {
      return this.newDatetime.day
    },
    hour () {
      return this.newDatetime.hour
    },
    minute () {
      return this.newDatetime.minute
    },
    dateFormatted () {
      return this.newDatetime.toLocaleString({
        month: 'long',
        day: 'numeric'
      })
    },
    minTime () {
      return (
        this.minDatetime &&
        this.minDatetime.year === this.year &&
        this.minDatetime.month === this.month &&
        this.minDatetime.day === this.day
      ) ? this.minDatetime.toFormat('HH:mm') : null
    },
    maxTime () {
      return (
        this.maxDatetime &&
        this.maxDatetime.year === this.year &&
        this.maxDatetime.month === this.month &&
        this.maxDatetime.day === this.day
      ) ? this.maxDatetime.toFormat('HH:mm') : null
    }
  },

  methods: {
    changeTimezone () {
      this.showTime = this.newDatetime.setZone(this.timezone).toFormat('HH:mm')
      this.showDate = this.newDatetime.setZone(this.timezone).toFormat('dd.MM.yyyy')
      this.$emit('end', this.newDatetime.setZone(this.timezone))
    },
    cancelTime () {
      this.step = 'date'
    },
    showTimePopup (event) {
      event.target.blur()
      this.step = 'time'
    },
    nextStep () {
      this.step = this.flow.next(this.step)
      this.timePartsTouched = []

      if (this.step === 'end') {
        this.$emit('confirm', this.newDatetime)
      }
    },
    showYear () {
      this.step = 'year'
      this.flow.diversion('date')
    },
    confirm () {
      this.nextStep()
    },
    cancel () {
      this.$emit('cancel')
    },
    onChangeYear (year) {
      this.newDatetime = this.newDatetime.set({ year })

      if (this.auto) {
        this.nextStep()
      }
    },
    onChangeDate (year, month, day) {
      this.newDatetime = this.newDatetime.set({ year, month, day })
      this.showDate = this.newDatetime.toFormat('dd.MM.yyyy')
      if (this.auto && !this.inline) {
        this.nextStep()
      }
      this.$emit('end', this.newDatetime)
    },
    onChangeTime ({ hour, minute, suffixTouched }) {
      if (suffixTouched) {
        this.timePartsTouched['suffix'] = true
      }

      if (Number.isInteger(hour)) {
        this.newDatetime = this.newDatetime.set({ hour })
        this.timePartsTouched['hour'] = true
      }

      if (Number.isInteger(minute)) {
        this.newDatetime = this.newDatetime.set({ minute })
        this.timePartsTouched['minute'] = true
      }

      this.showTime = this.newDatetime.toFormat('HH:mm')

      const goNext = this.auto && this.timePartsTouched['hour'] && this.timePartsTouched['minute'] && (
        this.timePartsTouched['suffix'] ||
        !this.use12Hour
      )

      this.$emit('end', this.newDatetime)

      if (goNext) {
        this.nextStep()
      }
    },
    onKeyDown (event) {
      switch (event.keyCode) {
        case KEY_ESC:
        case KEY_TAB:
          this.cancel()
          break

        case KEY_ENTER:
          this.nextStep()
          break
      }
    }
  }
}
</script>

<style>
.vdatetime-popup {
  min-width: 331px;
  padding-top: 24px;
  box-sizing: border-box;
  z-index: 1000;
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 340px;
  max-width: calc(100% - 30px);
  color: #444;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen", "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue", sans-serif;
  background: #fff;
  -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
  box-shadow: none;
  border: 1px solid #DBDBDB;
  border-radius: 3px;

  & * {
    box-sizing: border-box;
  }
}

.vdatetime-popup.inline {
  position: relative;
  left: auto;
  top: auto;
  transform: none;
}

.vdatetime-popup__header {
  padding: 15px 30px;
  background: #3f51b5;
  color: #fff;
  font-size: 32px;
}

.vdatetime-popup__year {
  display: block;
  font-weight: 300;
  font-size: 14px;
  opacity: 0.7;
  cursor: pointer;
  transition: opacity .3s;

  &:hover {
    opacity: 1;
  }
}

.vdatetime-popup__actions {
  padding: 0 20px 10px 30px;
  text-align: right;
}

.vdatetime-popup__actions__button {
  display: inline-block;
  border: none;
  padding: 10px 20px;
  background: transparent;
  font-size: 16px;
  color: #3f51b5;
  cursor: pointer;
  transition: color .3s;

  &:hover {
    color: #444;
  }
}

.vdatetime-popup__inline-bottom {
  display: flex;
  justify-content: space-between;
  font-size: 14px;
  padding: 24px;
  padding-bottom: 0;
}

.vdatetime-popup__inline-bottom._center {
  display: flex;
  justify-content: left;
  padding-bottom: 24px;
}

.vdatetime-popup__inline-bottom-date {
  max-width: 140px;
  font-size: 14px !important;
  border-radius: 3px;
  border: 1px solid #D2D2D2;
  padding: 8px 10px;
}

.vdatetime-popup__inline-bottom-time {
  max-width: 127px;
  font-size: 14px !important;
  border-radius: 3px;
  border: 1px solid #D2D2D2;
  padding: 8px 10px;
}

.vdatetime-popup__inline-bottom-timezone {
  border: none;
  background: none;
  outline: none;
  font-size: 14px !important;
  color: #989898;
}
</style>
