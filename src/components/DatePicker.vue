<template lang='pug'>
  .datepicker__wrapper(v-if='show' v-on-click-outside="hideDatepicker")
    .datepicker__close-button.-hide-on-desktop(v-if='isOpen' @click='hideDatepicker') ＋
    .datepicker__dummy-wrapper( @click='showDatepicker' :class="`${isOpen ? 'datepicker__dummy-wrapper--is-active' : ''}` ")
      input.datepicker__dummy-input.datepicker__input(
        ref="checkInInput"
        data-qa='datepickerInput'
        :class="`${isOpen && checkIn == null ? 'datepicker__dummy-input--is-active' : ''} ${singleDaySelection ? 'datepicker__dummy-input--single-date' : ''}`"
        v-model="checkInStr"
        :placeholder="i18n['check-in']"
        type="text"
        :disabled="keyboardFormats.length === 0 || !isDesktop"
        @keyup.self="setCheckInDateByInput"
        @keyup.enter="verifyCheckInDate"
        @blur="setCheckinBlur(isDesktop)"
        @focus="setCheckinFocus"
      )
      input.datepicker__dummy-input.datepicker__input(
        ref="checkOutInput"
        v-if='!singleDaySelection'
        :class="`${isOpen && checkOut == null && checkIn !== null ? 'datepicker__dummy-input--is-active' : ''}`"
        v-model="checkOutStr"
        :placeholder="i18n['check-out']"
        type="text"
        :disabled="keyboardFormats.length === 0 || !isDesktop"
        @keyup.self="setCheckOutDateByInput"
        @keyup.enter="verifyCheckOutDate(true)"
        @blur="setCheckoutBlur(isDesktop)"
        @focus="setCheckoutFocus"
      )
    button.datepicker__clear-button(type='button' @click='clearSelection()') ＋
    .datepicker( :class='`${ !isOpen ? "datepicker--closed" : "datepicker--open" }`')
      .-hide-on-desktop
        .datepicker__dummy-wrapper.datepicker__dummy-wrapper--no-border(
          @click='showDatepicker' :class="`${isOpen ? 'datepicker__dummy-wrapper--is-active' : ''}`"
          v-if="isOpen && screenSize != 'desktop'"
        )
          input.datepicker__dummy-input.datepicker__input(
            ref="checkInInputMobile"
            :class="`${isOpen && checkIn == null ? 'datepicker__dummy-input--is-active' : ''} ${singleDaySelection ? 'datepicker__dummy-input--single-date' : ''}`"
            v-model="checkInStr"
            :placeholder="i18n['check-in']"
            type="text"
            :disabled="keyboardFormats.length === 0"
            @keyup.self="setCheckInDateByInput"
            @keyup.enter="verifyCheckInDate"
            @blur="setCheckinBlur(false)"
            @focus="setCheckinFocus"
          )
          input.datepicker__dummy-input.datepicker__input(
            ref="checkOutInputMobile"
            v-if='!singleDaySelection'
            :class="`${isOpen && checkOut == null && checkIn !== null ? 'datepicker__dummy-input--is-active' : ''}`"
            v-model="checkOutStr"
            :placeholder="i18n['check-out']"
            type="text"
            :disabled="keyboardFormats.length === 0"
            @keyup.self="setCheckOutDateByInput"
            @keyup.enter="verifyCheckOutDate"
            @blur="setCheckoutBlur(false)"
            @focus="setCheckoutFocus"
          )
      .datepicker__inner
        div
        .datepicker__header
          span.datepicker__month-button.datepicker__month-button--prev.-hide-up-to-tablet(
            @click='showPreviousMonth'
            :class="{disabled: activeMonthIndex === 0}"
          )
          span.datepicker__month-button.datepicker__month-button--next.-hide-up-to-tablet(
            @click='showNextMonth'
            :class="{disabled: !isNextMonthAvailable}"
          )
        .datepicker__months(v-if='screenSize == "desktop"')
          div.datepicker__month(v-for='n in [0,1]'  v-bind:key='n')
            h1.datepicker__month-name(v-text='getMonth(months[activeMonthIndex+n].days[15].date)')
            .datepicker__week-row.-hide-up-to-tablet
              .datepicker__week-name(v-for='dayName in i18n["day-names"]' v-text='dayName')
            .square(v-for='day in months[activeMonthIndex+n].days'
              :class="{hidden: !day.belongsToThisMonth}"
              @mouseover='hoveringDate = day.date'
              @mouseleave='hoveringDate = null')
              Day(
                :options="$props"
                @dayClicked='handleDayClick($event)'
                :date='day.date'
                :sortedDisabledDates='sortedDisabledDates'
                :nextDisabledDate='nextDisabledDate'
                :activeMonthIndex='activeMonthIndex'
                :hoveringDate='hoveringDate'
                :tooltipMessage='tooltipMessage'
                :dayNumber='getDay(day.date)'
                :belongsToThisMonth='day.belongsToThisMonth'
                :checkIn='checkIn'
                :checkOut='checkOut'
              )
        div(v-if='screenSize !== "desktop" && isOpen')
          .datepicker__week-row
            .datepicker__week-name(v-for='dayName in this.i18n["day-names"]' v-text='dayName')
          .datepicker__months#swiperWrapper(ref="swiperWrapper")
            div.datepicker__month(v-for='(a, n) in months' v-bind:key='n' :ref="'month_' +  formatDate(months[n].days[15].date, 'YYYYMM')")
              h1.datepicker__month-name(v-text='getMonth(months[n].days[15].date)')
              .datepicker__week-row.-hide-up-to-tablet
                .datepicker__week-name(v-for='dayName in i18n["day-names"]' v-text='dayName')
              .square(v-for='(day, index) in months[n].days'
                :class="{hidden: !day.belongsToThisMonth}"
                @mouseover='hoveringDate = day.date'
                @mouseleave='hoveringDate = null'
                v-bind:key='index'
                )
                Day(
                  :options="$props"
                  @dayClicked='handleDayClick($event)'
                  :date='day.date'
                  :sortedDisabledDates='sortedDisabledDates'
                  :nextDisabledDate='nextDisabledDate'
                  :activeMonthIndex='activeMonthIndex'
                  :hoveringDate='hoveringDate'
                  :tooltipMessage='tooltipMessage'
                  :dayNumber='getDay(day.date)'
                  :belongsToThisMonth='day.belongsToThisMonth'
                  :checkIn='checkIn'
                  :checkOut='checkOut'
                )
</template>

<script>
import { directive as onClickOutside } from 'vue-on-click-outside';

import fecha from 'fecha';
import _ from 'lodash';

import Day from './Day.vue';
import Helpers from './helpers.js';

const defaulti18n = {
  night: 'Night',
  nights: 'Nights',
  'day-names': ['Sun', 'Mon', 'Tue', 'Wed', 'Thur', 'Fri', 'Sat'],
  'check-in': 'Check-in',
  'check-out': 'Check-out',
  'month-names': ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'],
};

export default {
  name: 'HotelDatePicker',

  directives: {
    'on-click-outside': onClickOutside
  },

  components: { Day, },

  props: {
    value: {
      type: String
    },
    startingDateValue: {
      default: null,
      type: Date
    },
    endingDateValue: {
      default: null,
      type: Date
    },
    format: {
      default: 'YYYY-MM-DD',
      type: String
    },
    startDate: {
      default: function() {
        return new Date();
      },
      type: [ Date, String ]
    },
    endDate: {
      default: Infinity,
      type: [ Date, String, Number ]
    },
    minNights: {
      default: 1,
      type: Number
    },
    maxNights: {
      default: null,
      type: Number
    },
    disabledDates: {
      default: function(){ return [] },
      type: Array
    },
    disabledDaysOfWeek: {
      default: function(){ return [] },
      type: Array
    },
    allowedRanges: {
      default: function(){ return [] },
      type: Array
    },
    hoveringTooltip: {
      default: true,
      type: [Boolean, Function]
    },
    tooltipMessage: {
      default: null,
      type: String
    },
    i18n: {
      default: () => defaulti18n,
      type: Object
    },
    enableCheckout: {
      default: false,
      type: Boolean
    },
    singleDaySelection: {
      default: false,
      type: Boolean
    },
    showYear: {
      default: false,
      type: Boolean
    },
    keyboardFormats: {
      default() {
        return [];
      },
      type: Array
    },
    open: {
      default: false,
      type: Boolean,
    }
  },

  data() {
    return {
      hoveringDate: null,
      checkIn: this.startingDateValue,
      checkOut: this.endingDateValue,
      currentDate: new Date(),
      months: [],
      activeMonthIndex: 0,
      nextDisabledDate: null,
      show: true,
      isOpen: false,
      xDown: null,
      yDown: null,
      xUp: null,
      yUp: null,
      sortedDisabledDates: null,
      checkInStr: null,
      checkOutStr: null,
      screenSize: this.handleWindowResize(),
      checkinFocus: false,
      checkoutFocus: false,
      monthRenderInterval: 6,
    };
  },

  computed: {
    isNextMonthAvailable() {
      const firstDayOfLastMonth = this.getFirstDayOfLastMonth();

      if (this.endDate !== Infinity) {
        if (
          fecha.format(firstDayOfLastMonth, "YYYYMM") ==
          fecha.format(new Date(this.endDate), "YYYYMM")
        ) {
          return false;
        }
      }
      return true;
    },
    isDesktop() {
      return this.screenSize == 'desktop';
    },
  },

  watch: {
    open(v) {
      this.isOpen = v;
    },
    isOpen (value) {
      if (this.screenSize !== 'desktop') {
        const bodyClassList = document.querySelector('body').classList;

        if (value) {
          bodyClassList.add('-overflow-hidden');
          this.$nextTick(this.registerTouchListeners);
        }
        else {
          bodyClassList.remove('-overflow-hidden');
          this.removeTouchListeners();
        }
      }
      this.$emit("toggle", value);
    },
    checkIn(newDate) {
      if (this.singleDaySelection) {
        this.hoveringDate = null;
        this.nextDisabledDate = null;
        this.parseDisabledDates();
      }
      this.$emit("checkInChanged", newDate )
    },
    checkOut(newDate) {
      if ( this.checkOut !== null) {
        this.hoveringDate = null;
        this.nextDisabledDate = null;
        this.parseDisabledDates();
      }

      this.$emit("checkOutChanged", newDate )
    },
    startingDateValue(v) {
      if (!this.checkinFocus) {
        this.checkIn = v;
        this.verifyCheckInDate();
      }
    },
    endingDateValue(v) {
      if (!this.checkoutFocus) {
        this.checkOut = v;
        this.verifyCheckOutDate();
      }
    },
  },

  methods: {
    ...Helpers,

    setCheckinBlur(isDesktop) {
      this.checkinFocus = false;
      if (isDesktop) {
        this.executeForDesktop(this.verifyCheckInDate);
      } else {
        this.verifyCheckInDate();
      }
    },
    setCheckoutBlur(isDesktop) {
      this.checkoutFocus = false;
      if (isDesktop) {
        this.executeForDesktop(this.verifyCheckOutDate);
      } else {
        this.verifyCheckOutDate();
      }
    },
    setCheckinFocus() {
      this.checkinFocus = true;
    },
    setCheckoutFocus() {
      this.checkoutFocus = true;
    },
    handleWindowResize() {
      let screenSizeInEm = window.innerWidth / parseFloat(getComputedStyle(document.querySelector('body'))['font-size']);

      if (screenSizeInEm < 31) {
        this.screenSize = 'smartphone';
      }
      else if (screenSizeInEm > 30 && screenSizeInEm < 57) {
        this.screenSize = 'tablet';
      }
      else if (screenSizeInEm > 56) {
        this.screenSize = 'desktop';
      }

      return this.screenSize;
    },

    onElementHeightChange(el, callback){
      let lastHeight = el.clientHeight;
      let newHeight = lastHeight;

      (function run(){
        newHeight = el.clientHeight;

        if( lastHeight !== newHeight ){
          callback();
        }

        lastHeight = newHeight;

        if( el.onElementHeightChangeTimer ) {
          clearTimeout(el.onElementHeightChangeTimer);
        }

        el.onElementHeightChangeTimer = setTimeout(run, 1000);
      })();
    },

    emitHeighChangeEvent() {
      this.$emit('heightChanged');
    },

    reRender(open) {
      this.show = false
      this.$nextTick(() => {
        this.show = true;
        if (open) {
          this.isOpen = true;
          this.$nextTick(() => {
            this.focusCheckIn();
          });
        }
      })
    },

    clearSelection(open = true) {
      this.hoveringDate = null;
      this.checkIn = null;
      this.checkInStr = null;
      this.checkOut = null;
      this.checkOutStr = null;
      this.nextDisabledDate = null;
      this.parseDisabledDates();
      if (open === true) {
        this.show = true;
        this.showDatepicker();
        this.focusCheckIn();
      }
      //this.reRender(true);
    },

    hideDatepicker() {
      this.isOpen = false;
    },

    showDatepicker() {
      if (!this.isOpen) {
        this.isOpen = true;
        this.$nextTick(() => {
          if (!this.checkOut && this.checkIn) {
            this.focusCheckOut();
          } else {
            this.focusCheckIn();
          }
          if (this.checkIn) {
            this.moveCalendarToTheDate(this.checkOut ? this.checkOut : this.checkIn);
          }
        });
      }
    },

    //doesn't used
    //toggleDatepicker() { this.isOpen = !this.isOpen; },

    getCheckInInput() {
      return this.screenSize == "desktop"
        ? this.$refs.checkInInput
        : this.$refs.checkInInputMobile;
    },

    getCheckOutInput() {
      return this.screenSize == "desktop"
        ? this.$refs.checkOutInput
        : this.$refs.checkOutInputMobile;
    },

    focusCheckIn() {
      const checkInInput = this.getCheckInInput();
      if (checkInInput) {
        checkInInput.focus();
      }
    },

    focusCheckOut() {
      const checkOutInput = this.getCheckOutInput();
      if (checkOutInput) {
        checkOutInput.focus();
      }
    },

    parseInputDate(str, isCheckin) {
      if (str && str.length > 0) {
        for (let i = 0; i < this.keyboardFormats.length; i++) {
          const format = this.keyboardFormats[i];
          const date = typeof format === 'function' ?
            format(str, isCheckin) :
            fecha.parse(str, format);
          if (date) {
            date.setHours(0,0,0,0);
            if (this.startDate) {
              this.startDate.setHours(0,0,0,0);
            }
            if (this.endDate && this.endDate !== Infinity) {
              this.endDate.setHours(0,0,0,0);
            }

            if (date >= this.startDate && date <= this.endDate) {
              if (!isCheckin && this.maxNights) {
                const timeDiff = Math.abs(date.getTime() - this.checkIn.getTime());
                const diffDays = Math.floor(timeDiff / (1000 * 3600 * 24));
                if (diffDays <= this.maxNights) {
                  return date;
                }
              } else {
                return date;
              }
            }
            if (isCheckin && date < this.startDate) {
              return this.startDate;
            }
            // if (!isCheckin) {
            //   if (this.maxNights) {
            //     const maxDate = new Date(this.checkIn.getTime());
            //     maxDate.setDate(maxDate.getDate() + this.maxNights);
            //     if (date > maxDate) {
            //       return maxDate;
            //     }
            //   } else if (date > this.endDate) {
            //     return this.endDate;
            //   }
            // }
          }
        }
        if (isCheckin && this.checkIn) {
          return this.startDate ? this.startDate : null;
        }
        if (!isCheckin && this.checkOut) {
          this.checkOut = null;
          this.onlyCheckInSetHandler();
        }
      }
      return false;
    },

    onlyCheckInSetHandler() {
      if (this.checkIn) {
        const allowedCheckoutDays = this.getAllowedCheckoutDays(
            this.checkIn,
            this.$props
          );

          this.nextDisabledDate = this.getNextDisabledDate(
            this.checkIn,
            this.$props,
            allowedCheckoutDays,
            this.sortedDisabledDates
          );

          this.moveCalendarToTheDate(this.checkIn);
      }
    },

    scrollToDate(date) {
      const monthEl = this.$refs[`month_${this.formatDate(date, 'YYYYMM')}`];
      if (monthEl && monthEl[0]) {
        const scrollTop = monthEl[0].offsetTop;
        this.$refs.swiperWrapper.scrollTop = scrollTop;
      }
    },

    moveCalendarToTheDate(date) {
      this.renderAllMonthesForDate(date);
      if (this.screenSize != 'desktop') {
        this.$nextTick(() => {
          this.scrollToDate(date);
        });
      }
    },

    executeForDesktop(fn) {
      if (this.screenSize == 'desktop') {
        fn();
      }
    },

    verifyCheckInDate() {
      if (this.checkIn) {
        this.checkInStr = this.formatDate(this.checkIn);
        this.$nextTick(() => {
          this.focusCheckOut();
        });
      } else if (this.checkInStr && this.checkInStr.length > 0) {
        this.clearSelection(false);
      }
    },

    verifyCheckOutDate(close) {
      if (this.checkOut) {
        this.checkOutStr = this.formatDate(this.checkOut);
        if (this.isOpen && close) {
          this.hideDatepicker();
        }
      } else if (this.checkOutStr && this.checkOutStr.length > 0) {
        this.checkOutStr = null;
        this.checkOut = null;
      }
    },

    setCheckInDateByInput() {
      const checkIn = this.parseInputDate(this.checkInStr, true);
      if (
        checkIn &&
        !this.checkIfDayIsDisabled(
          checkIn,
          this.checkIn,
          this.checkOut,
          this.sortedDisabledDates,
          this.$props
        )
      ) {
        this.checkIn = checkIn;

        this.onlyCheckInSetHandler();

        this.checkOut = null;
        this.checkOutStr = null;
      }
    },

    setCheckOutDateByInput() {
      const checkOut = this.parseInputDate(this.checkOutStr);
      if (
        checkOut &&
        (checkOut > this.checkIn || !this.checkIn) &&
        !this.checkIfDayIsDisabled(
          checkOut,
          this.checkIn,
          this.checkOut,
          this.sortedDisabledDates,
          this.$props
        )
      ) {
        this.checkOut = checkOut;

        const allowedCheckoutDays = this.getAllowedCheckoutDays(
          this.checkOut,
          this.$props
        );

        this.nextDisabledDate = this.getNextDisabledDate(
          this.checkOut,
          this.$props,
          allowedCheckoutDays,
          this.sortedDisabledDates
        );

        this.moveCalendarToTheDate(this.checkOut);
      }
    },

    handleDayClick(event) {
      if (this.checkIn == null && this.singleDaySelection == false) {
        this.checkIn = event.date;
        this.checkInStr = this.formatDate(this.checkIn);
        this.$nextTick(() => {
          this.focusCheckOut();
        });
      } else if (this.singleDaySelection == true) {
        this.checkIn = event.date;
        this.checkInStr = this.formatDate(this.checkIn);
        this.isOpen = false;
      } else if (this.checkIn !== null && this.checkOut == null) {
        this.checkOut = event.date;
        this.checkOutStr = this.formatDate(this.checkOut);
        this.isOpen = false;
      } else {
        this.checkOut = null;
        this.checkOutStr = null;
        this.checkIn = event.date;
        this.checkInStr = this.formatDate(this.checkIn);
      }

      this.nextDisabledDate = event.nextDisabledDate;
    },

    showPreviousMonth() {
      this.renderPreviousMonth();
    },

    showNextMonth() {
      this.renderNextMonth();
    },

    setCheckIn(date) { this.checkIn = date; },

    setCheckOut(date) { this.checkOut = date; },

    getDay(date) { return fecha.format(date, 'D') },

    getMonth(date) { return this.i18n["month-names"][fecha.format(date, 'M') - 1] + (this.showYear ? fecha.format(date, ' YYYY') : ''); },

    formatDate(date, format = false) {
      return fecha.format(date, format ? format : this.format);
    },

    createMonth(date){
      const firstSunday = this.getFirstSunday(date);

      let month = {
        days: []
      };

      for (let i = 0; i < 42; i++) {
        month.days.push({
          date: this.addDays(firstSunday, i),
          belongsToThisMonth: this.addDays(firstSunday, i).getMonth() === date.getMonth(),
          isInRange: false,
        });
      }

      this.months.push(month);
    },

    parseDisabledDates() {
      const sortedDates = [];

      for (let i = 0; i < this.disabledDates.length; i++) {
        sortedDates[i] = new Date(this.disabledDates[i]);
      }

      sortedDates.sort((a, b) =>  a - b );

      this.sortedDisabledDates = sortedDates;
    },
    registerTouchListeners() {
      if (this.screenSize != 'desktop') {
        if (this.endDate) {
          this.endRenderedDate = new Date();
          this.renderAdditionalMonthes();
          this.$refs.swiperWrapper.addEventListener('scroll', this.handleScroll);
        } else {
          document.addEventListener('touchstart', this.handleTouchStart, false);
          document.addEventListener('touchmove', this.handleTouchMove, false);
        }
      }
    },

    removeTouchListeners() {
      document.removeEventListener('touchstart', this.handleTouchStart);
      this.$refs.swiperWrapper.removeEventListener('scroll', this.handleScroll);
      document.removeEventListener('touchmove', this.handleTouchMove);
    }
  },

  beforeMount() {
    this.createMonth(new Date(this.startDate));
    this.createMonth(this.getNextMonth(new Date(this.startDate)));

    if (this.checkIn) {
      this.checkInStr = this.formatDate(this.checkIn);
    }
    if (this.checkOut) {
      this.checkOutStr = this.formatDate(this.checkOut);
    }
    this.parseDisabledDates();
  },

  mounted() {
    window.addEventListener('resize', this.handleWindowResize);

    this.onElementHeightChange(document.body, () => {
      this.emitHeighChangeEvent();
    });
  },

  destroyed() {
    window.removeEventListener('resize', this. handleWindowResize);
  }

};
</script>

<style lang="scss">
/* =============================================================
 * RESPONSIVE LAYOUT HELPERS
 * ============================================================*/
$tablet: '(min-width: 30em) and (max-width: 57em)';
$phone: '(max-width: 30em)';
$desktop: '(min-width: 57em)';
$tablet-up: '(min-width: 30em)';
$up-to-tablet: '(max-width: 57em)';
$extra-small-screen: '(max-width: 23em)';

@mixin device($device-widths) {
  @media screen and #{$device-widths} { @content }
}

.square {
  width: calc(100% / 7);
  float: left;
  &.hidden {
    visibility: hidden;
  }
}

*,
*::before,
*::after {
  box-sizing: border-box;
}

/* =============================================================
 * VARIABLES
 * ============================================================*/
$white: #fff;
$black:                #000;
$gray:                 #424b53;
$primary-text-color:   #35343d;
$lightest-gray:        #f3f5f8;
$primary-color: #00ca9d;
$primary-color: $primary-color;
$medium-gray: #999999;
$light-gray: #d7d9e2;
$dark-gray: #2d3047;

$font-small: 14px;

/* =============================================================
 * BASE STYLES
 * ============================================================*/

.datepicker {
  transition: all .2s ease-in-out;
  background-color: $white;
  color: $gray;
  font-size: 16px;
  line-height: 14px;
  overflow: hidden;
  left: 0;
  top: 48px;
  position: absolute;
  z-index: 10;

  &--closed {
    box-shadow: 0 15px 30px 10px rgba($black, 0);
    max-height: 0;
  }

  &--open {
    box-shadow: 0 15px 30px 10px rgba($black, .08);
    max-height: 900px;

    @include device($up-to-tablet) {
      box-shadow: none;
      height: 100%;
      left: 0;
      position: fixed;
      top: 0;
      width: 100%;
    }
  }

  &__wrapper {
    position: relative;
    display: inline-block;
    width: 100%;
    height: 48px;
    background: $white url('calendar_icon.regular.svg') no-repeat 17px center / 16px;
  }

  &__input {
    background: transparent;
    height: 48px;
    color: $primary-text-color;
    font-size: 12px;
    outline: none;
    padding: 4px 30px 2px;
    width: 100%;
    word-spacing: 5px;
    border: 0;

    &:focus {
      outline: none;
    }

    &::-webkit-input-placeholder,
    &::-moz-placeholder,
    &:-ms-input-placeholder,
    &:-moz-placeholder {
      color: $primary-text-color;
    }
  }

  &__dummy-wrapper {
    border: solid 1px $light-gray;
    cursor: pointer;
    display: block;
    float: left;
    width: 100%;
    height: 100%;

    &--no-border.datepicker__dummy-wrapper {
      margin-top: 15px;
      border: 0;
    }

    &--is-active {
      border: 1px solid $primary-color;
    }
  }

  &__dummy-input {
    color: $primary-text-color;
    padding-top: 0;
    font-size: $font-small;
    float: left;
    height: 48px;
    line-height: 3.1;
    text-align: left;
    text-indent: 5px;
    width: calc(50% + 4px);

    @include device($phone) {
      text-indent: 0;
      text-align: center;
    }

    &:first-child {
      background: transparent url('ic-arrow-right-datepicker.regular.svg') no-repeat right center / 8px;
      width: calc(50% - 4px);
      text-indent: 20px;
    }

    &--is-active { color: $primary-color; }
    &--is-active::placeholder { color: $primary-color; }
    &--is-active::-moz-placeholder { color: $primary-color; }
    &--is-active:-ms-input-placeholder { color: $primary-color; }
    &--is-active:-moz-placeholder { color: $primary-color; }
    &--single-date:first-child {
      width: 100%;
      background: none;
      text-align: left;
    }
  }

  &__month-day {
    visibility: visible;
    will-change: auto;
    text-align: center;
    margin: 0;
    border: 0;
    height: 40px;
    padding-top: 15px;

    &--invalid-range {
      background-color: rgba($primary-color, .3);
      color: $lightest-gray;
      cursor: not-allowed;
      position: relative;
    }

    &--invalid {
      color: $lightest-gray;
      cursor: not-allowed;
    }

    &--valid:hover,
    &--allowed-checkout:hover {
      background-color: $white;
      color: $primary-color;
      z-index: 1;
      position: relative;
      box-shadow: 0 0 10px 3px rgba($gray, .4);
    }

    &--disabled {
      color: $lightest-gray;
      cursor: not-allowed;
      pointer-events: none;
      position: relative;
    }

    &--selected {
      background-color: rgba($primary-color, .5);
      color: $white;

      &:hover {
        background-color: $white;
        color: $primary-color;
        z-index: 1;
        position: relative;
        box-shadow: 0 0 10px 3px rgba($gray, .4);
      }
    }

    &--today {
      background-color: $light-gray;
      color: $medium-gray;
    }

    &--first-day-selected,
    &--last-day-selected {
      background: $primary-color;
      color: $white;
    }

    &--allowed-checkout {
      color: $medium-gray;
    }

    &--out-of-range {
      color: $lightest-gray;
      cursor: not-allowed;
      position: relative;
      pointer-events: none;
    }

    &--valid {
      cursor: pointer;
      color: $medium-gray;
    }

    &--hidden {
      visibility: hidden;
      color: $white;
      pointer-events: none;
    }
  }

  &__month-button {
    background: transparent url('ic-arrow-right-green.regular.svg') no-repeat right center / 8px;
    cursor: pointer;
    display: inline-block;
    height: 60px;
    width: 60px;

    &--prev { transform: rotateY(180deg); }

    &--next { float: right; }

    &--locked {
      opacity: .2;
      cursor: not-allowed;
    }
  }

  &__inner {
    padding: 20px;
    float: left;

    @include device($up-to-tablet) { padding: 0; }
  }

  &__months {
    @include device($desktop) { width: 650px; }

    @include device($up-to-tablet) {
      margin-top: 92px;
      height: calc(100% - 92px);
      position: absolute;
      left: 0;
      top: 0;
      overflow: scroll;
      right: 0;
      bottom: 0;
    }

    &::before {
      background: $light-gray;
      bottom: 0;
      content: '';
      display: block;
      left: 50%;
      position: absolute;
      top: 0;
      width: 1px;

      @include device($up-to-tablet) { display: none; }
    }
  }

  &__month {
    font-size: 12px;
    float: left;
    width: 50%;
    padding-right: 10px;

    @include device($up-to-tablet) {
      width: 100%;
      padding-right: 0;
      padding-top: 45px;

      &:last-of-type {
        padding-top: 0;
        padding-left: 0;
        margin-top: 35px;
        margin-bottom: 200px;
      }
    }

    @include device($tablet-up) {
      &:last-of-type {
        padding-right: 0;
        padding-left: 10px;
      }
    }
  }

  &__month-caption {
    height: 2.5em;
    vertical-align: middle;
  }

  &__month-name {
    font-size: 16px;
    font-weight: 500;
    margin-top: -40px;
    padding-bottom: 17px;
    pointer-events: none;
    text-align: center;

    @include device($up-to-tablet) {
      margin-top: -25px;
      margin-bottom: 0;
      position: absolute;
      width: 100%;
      left: 0;
    }
  }

  &__week-days {
    height: 2em;
    vertical-align: middle;
  }

  &__week-row {
    border-bottom: 5px solid $white;
    height: 38px;

    @include device($up-to-tablet) {
      box-shadow: 0 13px 18px -8px rgba($black, .07);
      height: 25px;
      left: 0;
      top: 65px;
      position: absolute;
      width: 100%;
    }
  }

  &__week-name {
    width: calc(100% / 7);
    float: left;
    font-size: 12px;
    font-weight: 400;
    color: $medium-gray;
    text-align: center;
  }

  &__close-button {
    appearence: none;
    background: transparent;
    border: 0;
    color: $primary-color;
    cursor: pointer;
    font-size: 21px;
    font-weight: bold;
    margin-top: 0;
    outline: 0;
    z-index: 10000;
    position: fixed;
    left: 7px;
    top: 5px;
    transform: rotate(45deg);
  }

  &__clear-button {
    appearence: none;
    background: transparent;
    border: 0;
    color: $primary-color;
    cursor: pointer;
    font-size: 25px;
    font-weight: bold;
    height: 40px;
    margin-bottom: 0;
    margin-left: 0;
    margin-right: -2px;
    margin-top: 4px;
    outline: 0;
    padding: 0;
    position: absolute;
    right: 0;
    top: 0;
    transform: rotate(45deg);
    width: 40px;
  }

  &__tooltip {
    background-color: $dark-gray;
    border-radius: 2px;
    color: $white;
    font-size: 11px;
    margin-left: 5px;
    margin-top: -22px;
    padding: 5px 10px;
    position: absolute;
    z-index: 50;

    &:after {
      border-left: 4px solid transparent;
      border-right: 4px solid transparent;
      border-top: 4px solid $dark-gray;
      bottom: -4px;
      content: '';
      left: 50%;
      margin-left: -4px;
      position: absolute;
    }
  }
}

// Modifiers

.-overflow-hidden { overflow: hidden; }

.-is-hidden { display: none; }

.-hide-up-to-tablet {
  @include device($up-to-tablet) {
    display: none;
  }
}

.-hide-on-desktop {
  @include device($desktop) {
    display: none;
  }
}

</style>
