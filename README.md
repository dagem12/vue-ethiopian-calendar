# Ethiopian Calendar Vue Component

A Vue 2 component for displaying and selecting dates in the Ethiopian calendar. Supports both Amharic and English languages with customizable styling.

## Installation

```bash
npm install vue-ethiopian-calendar
# or
yarn add vue-ethiopian-calendar
```

## Basic Usage

### 🌍 Global Registration (Recommended for app-wide usage)

Register the component globally in your main application file:

```javascript
// main.js
import Vue from 'vue';
import EthiopianCalendar from 'vue-ethiopian-calendar';
import 'vue-ethiopian-calendar/dist/EthiopianCalendar.css';

// Register globally - available in all components
Vue.component('EthiopianCalendar', EthiopianCalendar);

new Vue({
  render: h => h(App),
}).$mount('#app');
```

Now you can use it anywhere in your app without importing:
```vue
<template>
  <EthiopianCalendar @date-selected="handleDateChange" language="am"/>
</template>
```

### 🎯 Specific Component Usage 

Import directly in your Vue component:

```vue
<template>
  <div class="date-picker">
    <h3>Select Ethiopian Date</h3>
    <EthiopianCalendar 
      language="am"
      @date-selected="logDate"
    />
  </div>
</template>

<script>
import EthiopianCalendar from 'ethiopian-calendar-vue2';
import 'ethiopian-calendar-vue2/dist/EthiopianCalendar.css';

export default {
  name: 'MyDatePicker',
  components: {
    EthiopianCalendar
  },
  methods: {
    logDate({ ethiopian, gregorian }) {
      console.log('Ethiopian:', ethiopian);
      console.log('Gregorian:', gregorian);
    }
  }
};
</script>

<style>
.date-picker {
  max-width: 400px;
  margin: 0 auto;
}
</style>
```






## Available Props

| Prop            | Type   | Default    | Description |
|---------------|--------|------------|-------------|
| `language`       | String | "am"       | Calendar language (`"am"` for Amharic or `"en"` for English) |
| `primaryColor`   | String | `"#ffffff"` | Primary theme color |
| `hoverColor`     | String | `"#ffe08a"` | Hover color |
| `todayColor`     | String | `"#1fe08a"` | Today indicator color |
| `selectedColor`  | String | `"#ccdefe"` | Selected date color |
| `textColor`      | String | `"#2d3748"` | Text color |
| `backgroundColor`| String | `"#ffffff"` | Background color |
| `borderColor`    | String | `"#ffffff"` | Border color |
| `headerBg`       | String | `"#f7fafc"` | Header background color |
| `weekdayColor`   | String | `"#718096"` | Weekday text color |
| `fontStack`      | String | System fonts | Custom font stack |

## Events

### `date-selected`

Emitted when a date is selected. Returns an object with:

```javascript
{
  ethiopianDate: {
    year: Number,  // Ethiopian year
    month: Number, // Ethiopian month (1-13)
    day: Number    // Ethiopian day
  },
  gregorianDate: {
    year: Number,  // Gregorian year
    month: Number, // Gregorian month (1-12)
    day: Number    // Gregorian day
  }
}
```

## Example with All Props

```vue
<EthiopianCalendar
  @date-selected="handleDateSelection"
  language="am"
  primaryColor="#4a86e8"
  hoverColor="#356dca"
  todayColor="#ffe08a"
  selectedColor="#4a86e8"
  textColor="#2d3748"
  backgroundColor="#ffffff"
  borderColor="#e2e8f0"
  headerBg="#f7fafc"
  weekdayColor="#718096"
  fontStack="'Custom Font', sans-serif"
/>
```

## Navigation Methods

- **Month navigation** (`◀ ▶` buttons)
- **Year navigation** (`« »` buttons)
- **Date selection** (clicking a day)

## Styling Notes

- All colors can be customized via props.
- Responsive design for mobile devices.
- Clean, modern UI with subtle animations.

## TypeScript Support

The component includes TypeScript definitions for all props and emitted events.

## Browser Support

✅ Chrome  
✅ Firefox  
✅ Safari  
✅ Edge  
✅ IE11 (with polyfills)

---

### Author
**Dagem Getachew Asefa**

### License
[MIT](LICENSE)

