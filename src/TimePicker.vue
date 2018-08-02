<template>
  <div class="timepicker">
    <input id="timepicker-input" v-model="displayTime" type="text" class="input-time" @click="open" @blur="validate()" @keyup="isSubmit">
    <ul v-if="dropdownOpen" ref="timeDropdown" class="time-dropdown">
      <li v-for="(time, i) in timeArray" :key="i" class="time-dropdown-item" @mousedown.prevent @click="selectTime(time)">
        {{ time.hours }}:{{ time.minutes }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'Timepicker',
  props: {
    enabledTimes: Object,
    value: Object
  },
  data() {
    return {
      validatedSelectedTime: null,
      interval: 15,
      dropdownOpen: false,
      selectedTime: null,
      displayTime: null,
      minTime: null
    }
  },
  computed: {
    timeArray() {
      const timeArray = []

      const fromHours = this.enabledTimes.from
        ? this.enabledTimes.from.hours
        : 0
      const fromMinutes = this.enabledTimes.from
        ? this.enabledTimes.from.minutes
        : 0
      const toHours = this.enabledTimes.to ? this.enabledTimes.to.hours : 24
      // let toMinutes = this.enabledTimes.to ? this.enabledTimes.to.minutes : 0;

      for (let i = fromHours; i < toHours; ++i) {
        for (
          let j = i == fromHours ? Math.ceil(fromMinutes / this.interval) : 0;
          j < 60 / this.interval;
          ++j
        ) {
          const time = {
            hours: (i < 10 ? '0' : '') + i % 24,
            minutes: j * this.interval || '00'
          }
          timeArray.push(time)
        }
      }
      // this.minTime = timeArray[0];
      return timeArray
    }
  },
  watch: {
    value(value) {
      this.setValue(value)
    }
  },
  mounted() {
    this.init()
  },
  methods: {
    init() {
      if (this.value) {
        this.setValue(this.value)
      }
    },
    setValue(time) {
      this.selectedTime = time
      this.setDisplayTime(this.selectedTime)
    },
    isSubmit(e) {
      if (e.keyCode === 13) {
        this.validate()
      }
    },
    validate() {
      const timeRegex = new RegExp('^([01]\\d|2[0-3]):?([0-5]\\d)$')
      if (this.displayTime.match(timeRegex)) {
        this.displayTime = this.displayTime.replace(':', '')
        const newTime = {
          hours: this.displayTime.substr(0, 2),
          minutes: this.displayTime.substr(2)
        }
        if (this.isWithinValidTimeSpan(newTime)) {
          this.selectedTime = newTime
        } else {
          this.selectedTime = this.minTime
        }
        this.validatedSelectedTime = this.selectedTime
      }
      this.setDisplayTime(this.selectedTime)
      this.close()
      this.$emit('selected', this.selectedTime)
    },
    isWithinValidTimeSpan(newTime) {
      if (!this.enabledTimes) {
        return true
      }
      let isValid = false
      if (
        newTime.hours > this.enabledTimes.from.hours ||
        (newTime.hours == this.enabledTimes.from.hours &&
          newTime.minutes >= this.enabledTimes.from.minutes)
      ) {
        if (
          newTime.hours < this.enabledTimes.to.hours ||
          (newTime.hours == this.enabledTimes.to.hours &&
            newTime.minutes <= this.enabledTimes.to.minutes)
        ) {
          isValid = true
        }
      }
      return isValid
    },
    preventTyping(event) {
      event.preventDefault()
      return false
    },
    selectTime(time) {
      this.selectedTime = time
      this.setDisplayTime(this.selectedTime)
      this.validate()
      this.close()
    },
    setDisplayTime(time) {
      if (time) {
        this.displayTime = `${((time.hours < 10 ? '0' : '') + time.hours).slice(
          -2
        )}:${((time.minutes < 10 ? '0' : '') + time.minutes).slice(-2)}`
      }
    },
    open() {
      this.dropdownOpen = true
    },
    close() {
      this.dropdownOpen = false
    }
  }
}
</script>

<style lang="scss" scoped>
.timepicker {
  position: relative;
}
.time-dropdown {
  position: absolute;
  top: 100%;
  list-style-type: none;
  margin: 0;
  padding: 0;
  background: #fff;
  border-radius: 4px;
  overflow-y: auto;
  overflow-x: hidden;
  max-height: 220px;
  width: 100%;
  z-index: 2;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.33);
}

.time-dropdown-item {
  font-weight: 700;
  color: var(--gray-darker);
  padding: var(--padding-default);
  &:hover {
    background: var(--color-accent-light);
    color: var(--color-accent);
    cursor: pointer;
  }
}
</style>
