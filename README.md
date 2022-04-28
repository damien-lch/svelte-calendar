# 🗓️ Svelte Calendar
A simple Svelte calendar that can act as a date picker.
# Props
*Please note that all dates are on the **YYYY-MM-DD** format.*

| Name               | Description                                            | Type     | Default | isReactive |
| ------------------ | ------------------------------------------------------ | -------- | ------- | ---------- |
| mode               | The mode of the calendar : display, single or multiple | String   | single  | False      |
| locale             | The locale of the calendar (FR / EN)                   | String   | en      | False      |
| minDate            | The minimun allowed date of the calendar               | String   | null    | False      |
| maxDate            | The maximum allowed date of the calendar               | String   | null    | False      |
| disabledDates      | Array of disabled dates                                | String[] | []      | False      |
| maxSelectableDates | Maximum of date selectable in multiple mode            | Number   | 2       | False      |
| selectedDate       | Date selected on single mode                           | String   | null    | True       |
| selectedDates      | Array of selected dates on multiple mode               | String[] | []      | True       |
# Behavior
- selectedDate and selectedDates are reset when mode is updated.