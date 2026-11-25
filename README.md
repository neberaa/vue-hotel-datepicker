# vue-hotel-datepicker

A Vue.js date-range picker for hotel and accommodation booking flows. This repository contains a maintained/extended copy of the upstream component, with additional fixes contributed in the context of a production hospitality project.

> **Contribution and attribution:** This is not an original project created from scratch by the repository owner. The component originated with [krystalcampioni/vue-hotel-datepicker](https://github.com/krystalcampioni/vue-hotel-datepicker) and its upstream contributors. My documented contributions in this copy include fixes for single-day date selection, including mobile behaviour, and closing the picker after a single-day selection. See the repository commit history for the implementation trail. Please retain the upstream project's attribution and licensing notices when reusing this code.

## Overview

A responsive date range picker for Vue.js that displays the number of nights selected and supports configurable check-in/check-out rules, localisation and other booking-oriented options.

![demo gif](https://github.com/krystalcampioni/vue-hotel-datepicker/blob/master/demo.gif?raw=true)

## Demo

[Live demo](https://krystalcampioni.github.io/vue-hotel-datepicker/)

## Installation

```bash
npm install vue-hotel-datepicker --save
```

```javascript
import HotelDatePicker from 'vue-hotel-datepicker'

export default {
  components: {
    HotelDatePicker,
  },
}
```

```html
<HotelDatePicker />
```

## Props / options

### format

- Type: `String`
- Default: `YYYY-MM-DD`

The date format string.

### startDate

- Type: `Date` or `String`
- Default: `new Date()`

The start view date. All dates before this date will be disabled.

### startingDateValue

- Type: `Date`
- Default: `null`

The initial value of the start date.

### endDate

- Type: `Date`, `String` or `Boolean`
- Default: `false`

The end view date. All dates after this date will be disabled.

### endingDateValue

- Type: `Date`
- Default: `null`

The initial value of the end date.

### minNights

- Type: `Number`
- Default: `1`

Minimum nights required to select a range of dates.

### maxNights

- Type: `Number`
- Default: `0`

Maximum nights required to select a range of dates.

### disabledDates

- Type: `Array`
- Default: `[]`

An array of date strings in `YYYY-MM-DD` format. All dates in the list will be disabled.

### disabledDaysOfWeek

- Type: `Array`
- Default: `[]`

An array of day names, for example `['Monday', 'Tuesday', 'Wednesday']`. All listed days will be disabled.

### allowedRanges

- Type: `Array`
- Default: `[]`

An array of numbers. For example, `[7, 10, 14]`. After selecting the start date, the calendar will allow checkout only 7, 10 or 14 days later.

### enableCheckout

- Type: `Boolean`
- Default: `false`

If `true`, allows checkout on a disabled date.

### hoveringTooltip

- Type: `Boolean` or `Function`
- Default: `true`

Shows a tooltip with the number of nights when hovering over a date.

### tooltipMessage

- Type: `String`
- Default: `null`

If provided, overrides the default tooltip text (for example, "X nights"). The original component also permits HTML in this string; only use trusted content.

### singleDateSelection

- Type: `Boolean`
- Default: `false`

Enables selecting a single date instead of a date range.

### showYear

- Type: `Boolean`
- Default: `false`

Shows the year next to the month.

### keyboardFormats

- Type: `Array` of `String` or `Function`
- Default: `[]`

Enables direct keyboard input. Parsing is executed by `fecha.parse()` with the provided format or function. The first valid format is used. See the [fecha issue](https://github.com/taylorhakes/fecha/issues/22) for a related parsing consideration.

### i18n

- Type: `Object`

Default:

```js
i18n: {
  night: 'Night',
  nights: 'Nights',
  'day-names': ['Sun', 'Mon', 'Tue', 'Wed', 'Thur', 'Fri', 'Sat'],
  'check-in': 'Check-in',
  'check-out': 'Check-Out',
  'month-names': ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'],
}
```

## API

- `hideDatepicker()` — hide the datepicker
- `showDatepicker()` — show the datepicker
- `toggleDatepicker()` — toggle visibility

## Events

- `checkInChanged` — emitted when a new check-in date is selected; includes the selected date as payload
- `checkOutChanged` — emitted when a new check-out date is selected; includes the selected date as payload
- `toggle` — emitted when the datepicker is shown or hidden

## Credits

The original component was built as a Vue wrapper for [Hotel Datepicker](https://github.com/benitolopez/hotel-datepicker) by @benitolopez. The upstream Vue component is maintained at [krystalcampioni/vue-hotel-datepicker](https://github.com/krystalcampioni/vue-hotel-datepicker). This repository includes subsequent fixes and extensions; consult the commit history to distinguish those contributions from the original implementation.
