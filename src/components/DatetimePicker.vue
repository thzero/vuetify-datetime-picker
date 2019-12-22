<template>
  <v-dialog v-model="display" :width="dialogWidth">
    <template v-slot:activator="{ on }">
      <v-text-field
        v-bind="textFieldProps"
        :disabled="disabled"
        :loading="loading"
        :label="label"
        :value="formattedDateTime"
        v-on="on"
        readonly
      >
        <template v-slot:progress>
          <slot name="progress">
            <v-progress-linear color="primary" indeterminate absolute height="2"></v-progress-linear>
          </slot>
        </template>
      </v-text-field>
    </template>

    <v-card>
      <v-card-text class="px-0 py-0">
        <v-tabs fixed-tabs v-model="activeTab">
          <v-tab key="calendar">
            <slot name="dateIcon">
              <v-icon>event</v-icon>
            </slot>
          </v-tab>
          <v-tab key="timer" :disabled="dateSelected">
            <slot name="timeIcon">
              <v-icon>access_time</v-icon>
            </slot>
          </v-tab>
          <v-tab-item key="calendar">
            <v-date-picker v-model="date" v-bind="datePickerProps" @input="showTimePicker" full-width></v-date-picker>
          </v-tab-item>
          <v-tab-item key="timer">
            <v-time-picker
              ref="timer"
              class="v-time-picker-custom"
              v-model="time"
              v-bind="timePickerProps"
              full-width
            ></v-time-picker>
          </v-tab-item>
        </v-tabs>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <slot name="actions" :parent="this">
          <v-btn color="primary lighten-1" text @click.native="clearHandler">{{ clearText }}</v-btn>
          <v-btn color="primary lighten-1" text @click.native="cancelHandler">{{ cancelText }}</v-btn>
          <v-btn color="green darken-1" text @click="okHandler">{{ okText }}</v-btn>
        </slot>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import moment from 'moment'
// import { format, parse } from 'date-fns'

const DEFAULT_CANCEL_TEXT = 'CANCEL'
const DEFAULT_CLEAR_TEXT = 'CLEAR'
const DEFAULT_DATE = ''
const DEFAULT_DATE_FORMAT = 'YYYY-MM-DD'
// const DEFAULT_DATE_FORMAT = 'yyyy-MM-dd'
const DEFAULT_DIALOG_WIDTH = 340
const DEFAULT_OK_TEXT = 'OK'
const DEFAULT_TIME = '00:00:00'
const DEFAULT_TIME_FORMAT = 'HH:mm:ss'
// const DEFAULT_TIME_FORMAT = 'HH:mm:ss'
const DEFAULT_TIME_OUTPUT_FORMAT = 'HH:mm'
const OUTPUT_TYPE_DATE = 'date'
const OUTPUT_TYPE_TIMESTAMP = 'timestamp'

export default {
  name: 'v-datetime-picker',
  // model: {
  //   prop: 'datetime',
  //   event: 'input'
  // },
  props: {
    value: {
      type: [Date, String, Number],
      default: null
    },
    cancelText: {
      type: String,
      default: DEFAULT_CANCEL_TEXT
    },
    clearText: {
      type: String,
      default: DEFAULT_CLEAR_TEXT
    },
    dateFormat: {
      type: String,
      default: DEFAULT_DATE_FORMAT
    },
    datePickerProps: {
      type: Object
    },
    disabled: {
      type: Boolean
    },
    dialogWidth: {
      type: Number,
      default: DEFAULT_DIALOG_WIDTH
    },
    label: {
      type: String,
      default: ''
    },
    loading: {
      type: Boolean
    },
    okText: {
      type: String,
      default: DEFAULT_OK_TEXT
    },
    outputType: {
      type: String,
      default: OUTPUT_TYPE_DATE,
      validator: (val) => [OUTPUT_TYPE_DATE, OUTPUT_TYPE_TIMESTAMP].includes(val)
    },
    textFieldProps: {
      type: Object
    },
    timeFormat: {
      type: String,
      default: DEFAULT_TIME_OUTPUT_FORMAT
    },
    timePickerProps: {
      type: Object
    }
  },
  data: () => ({
    display: false,
    activeTab: 0,
    date: DEFAULT_DATE,
    internalOutputType: OUTPUT_TYPE_DATE,
    time: DEFAULT_TIME
  }),
  mounted() {
    this.init()
  },
  computed: {
    dateSelected() {
      return !this.date
    },
    dateTimeFormat() {
      return this.dateFormat + ' ' + this.timeFormat
    },
    defaultDateTimeFormat() {
      return DEFAULT_DATE_FORMAT + ' ' + DEFAULT_TIME_FORMAT
    },
    formattedDateTime() {
      return this.formatDateTime(this.selectedDatetime)
    },
    selectedDatetime() {
      if (this.date && this.time) {
        let datetimeString = this.date + ' ' + this.time
        if (this.time.length === 5)
          datetimeString += ':00'

        let date = moment(datetimeString, this.defaultDateTimeFormat)
        // let date = parse(datetimeString, this.defaultDateTimeFormat, new Date())
        return date
      }

      return null
    }
  },
  methods: {
    cancelHandler() {
      this.resetPicker()
    },
    clearHandler() {
      this.resetPicker()
      this.date = DEFAULT_DATE
      this.time = DEFAULT_TIME
      this.$emit('input', null)
    },
    formatDateTime(selectedDatetime) {
      return selectedDatetime ? selectedDatetime.format(this.dateTimeFormat) : ''
      //return selectedDatetime ? format(selectedDatetime, this.dateTimeFormat) : ''
    },
    init() {
      if (!this.value)
        return

      let initDateTime
      if (this.datetime instanceof Date)
        initDateTime = moment(this.value)
      else if (typeof this.value === 'number' || this.value instanceof Number)
        initDateTime = moment(this.value)
      else if (typeof this.value === 'string' || this.value instanceof String) {
        initDateTime = moment(this.value, this.dateTimeFormat)
        // see https://stackoverflow.com/a/9436948
        // initDateTime = parse(this.datetime, this.dateTimeFormat, new Date())
      }

      this.date = initDateTime.format(DEFAULT_DATE_FORMAT)
      this.time = initDateTime.format(DEFAULT_TIME_FORMAT)
      // this.date = format(initDateTime, DEFAULT_DATE_FORMAT)
      // this.time = format(initDateTime, DEFAULT_TIME_FORMAT)
    },
    okHandler() {
      this.$emit('input', this.internalOutputType == OUTPUT_TYPE_DATE ? this.formatDateTime(this.selectedDatetime) : this.timestamp())
      this.resetPicker()
    },
    resetPicker() {
      this.display = false
      this.activeTab = 0
      if (this.$refs.timer)
        this.$refs.timer.selectingHour = true
    },
    timestamp() {
      return this.selectedDatetime ? moment(this.selectedDatetime).valueOf() : 0
    },
    showTimePicker() {
      this.activeTab = 1
    }
  },
  watch: {
    outputType(newVal) {
      this.internalOutputType = newVal
    },
    value() {
      this.init()
    }
  }
}
</script>
