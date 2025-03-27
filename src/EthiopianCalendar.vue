<template>
    <div class="calendar-container" :style="cssVariables">
      <div class="calendar-header">
        <button @click="prevYear">«</button>
        <button @click="prevMonth">◀</button>
        <span>{{ monthNames[selectedMonth - 1] }} {{ selectedYear }}</span>
        <button @click="nextMonth">▶</button>
        <button @click="nextYear">»</button>
      </div>
  
      <div class="calendar-weekdays">
        <div v-for="day in weekdayNames" :key="day">{{ day }}</div>
      </div>
  
      <div class="calendar-days">
        <div v-for="blank in firstDayOffset" :key="'blank' + blank" class="calendar-day empty"></div>
        <div v-for="day in daysInMonth" :key="day" class="calendar-day"
             :class="{ selected: day === selectedDay}"
             @click="selectDate(day)">
          {{ day }}
        </div>
      </div>
      <div class="calendar-footer">
  {{ selectedDay }} {{ monthNames[selectedMonth - 1] }} {{ selectedYear }}
</div>
    </div>
  </template>
  
  <script>
  export default {
    props: {
      language: {
        type: String,
        default: "am",
        validator: value => ["am", "en"].includes(value),
      },
      primaryColor: {
      type: String,
      default: '#ffffff'
    },
    hoverColor: {
      type: String,
      default: '#ffe08a'
    },
    todayColor: {
      type: String,
      default: '#1fe08a'
    },
    selectedColor: {
      type: String,
      default: '#ccdefe'
    },
    textColor: {
      type: String,
      default: '#2d3748'
    },
    backgroundColor: {
      type: String,
      default: '#ffffff'
    },
    borderColor: {
      type: String,
      default: '#ffffff'
    },
    headerBg: {
      type: String,
      default: '#f7fafc'
    },
    weekdayColor: {
      type: String,
      default: '#718096'
    },
    fontStack: {
      type: String,
      default: "-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Noto Sans Ethiopic', sans-serif"
    }
    },
    data() {
      // Initialize with current Ethiopian date
      const today = new Date();
      const ethDate = this.gregorianToEthiopian(today.getFullYear(), today.getMonth() + 1, today.getDate());
      
      return {
        selectedYear: ethDate.year,
        selectedMonth: ethDate.month,
        selectedDay: ethDate.day,
        months: {
          am: ["መስከረም", "ጥቅምት", "ኅዳር", "ታኅሳስ", "ጥር", "የካቲት", "መጋቢት", "ሚያዝያ", "ግንቦት", "ሰኔ", "ሀምሌ", "ነሐሴ", "ጳጉሜ"],
          en: ["Meskerem", "Tikimt", "Hidar", "Tahsas", "Tir", "Yekatit", "Megabit", "Miyazya", "Ginbot", "Sene", "Hamle", "Nehase", "Pagume"]
        },
        weekdays: {
          am: ["እሑድ", "ሰኞ", "ማክሰኞ", "ረቡዕ", "ሐሙስ", "ዓርብ", "ቅዳሜ"],
          en: ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"]
        }
      };
    },
    computed: {
      monthNames() {
        return this.months[this.language];
      },
      weekdayNames() {
        return this.weekdays[this.language];
      },
      daysInMonth() {
        if (this.selectedMonth === 13) {
          return this.isEthiopianLeapYear(this.selectedYear) ? 6 : 5;
        }
        return 30;
      },
      firstDayOffset() {
        const gregDate = this.ethiopianToGregorian(this.selectedYear, this.selectedMonth, 1);
        return gregDate.getDay(); // 0=Sunday, 1=Monday, etc.
      },
      cssVariables() {
      return {
        '--primary-color': this.primaryColor,
        '--hover-color': this.hoverColor,
        '--today-color': this.todayColor,
        '--selected-color': this.selectedColor,
        '--text-color': this.textColor,
        '--background-color': this.backgroundColor,
        '--border-color': this.borderColor,
        '--header-bg': this.headerBg,
        '--weekday-color': this.weekdayColor,
        '--font-stack': this.fontStack
      };
    }
    },
    methods: {
      startDayOfEthiopian(year) {
        let newYearDay = Math.floor(year / 100) - Math.floor(year / 400) - 4;
        if ((year - 1) % 4 === 3) {
          newYearDay += 1;
        }
        return newYearDay;
      },
  
      isEthiopianLeapYear(year) {
        return (year) % 4 === 3;
      },
  
      ethiopianToGregorian(eYear, eMonth, eDay) {
        // Prevent incorrect input
        if (eYear <= 0 || eMonth <= 0 || eDay <= 0) {
          throw new Error("Malformed input can't be converted.");
        }
  
        // Ethiopian new year in Gregorian calendar
        const newYearDay = this.startDayOfEthiopian(eYear);
  
        // September (Ethiopian) sees 7y difference
        let gregorianYear = eYear + 7;
  
        // Number of days in gregorian months starting with September (index 1)
        const gregorianMonths = [0, 30, 31, 30, 31, 31, 28, 31, 30, 31, 30, 31, 31, 30];
  
        // Check if next Gregorian year is leap year
        const nextYear = gregorianYear + 1;
        if ((nextYear % 4 === 0 && nextYear % 100 !== 0) || nextYear % 400 === 0) {
          gregorianMonths[6] = 29; // February has 29 days
        }
  
        // Calculate number of days up to that date
        let until = ((eMonth - 1) * 30) + eDay;
        if (until <= 37 && eYear <= 1575) {  // mysterious rule
          until += 28;
          gregorianMonths[0] = 31;
        } else {
          until += newYearDay - 1;
        }
  
        // If Ethiopian year is leap year, paguemain has six days
        if (eYear - 1 % 4 === 3) {
          until += 1;
        }
  
        // Calculate month and date incremently
        let m = 0;
        let gregorianDate = 0;
        for (m = 0; m < gregorianMonths.length; m++) {
          if (until <= gregorianMonths[m]) {
            gregorianDate = until;
            break;
          } else {
            until -= gregorianMonths[m];
          }
        }
  
        // If m > 4, we're already on next Gregorian year
        if (m > 4) {
          gregorianYear += 1;
        }
  
        // Gregorian months ordered according to Ethiopian
        const order = [8, 9, 10, 11, 12, 1, 2, 3, 4, 5, 6, 7, 8, 9];
        const gregorianMonth = order[m];
  
        return new Date(gregorianYear, gregorianMonth - 1, gregorianDate);
      },
  
      gregorianToEthiopian(gYear, gMonth, gDay) {
        // Prevent incorrect input
        if (gYear <= 0 || gMonth <= 0 || gDay <= 0) {
          throw new Error("Malformed input can't be converted.");
        }
  
        // Date between 5 and 14 of May 1582 are invalid
        if (gMonth === 5 && gDay >= 5 && gDay <= 14 && gYear === 1582) {
          throw new Error("Invalid Date between 5-14 May 1582.");
        }
  
        // Number of days in gregorian months starting with January (index 1)
        const gregorianMonths = [0, 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];
  
        // Check for Gregorian leap year
        if ((gYear % 4 === 0 && gYear % 100 !== 0) || gYear % 400 === 0) {
          gregorianMonths[2] = 29;
        }
  
        // September sees 8y difference
        let ethiopianYear = gYear - 8;
  
        // Number of days in ethiopian months starting with January (index 1)
        const ethiopianMonths = [0, 30, 30, 30, 30, 30, 30, 30, 30, 30, 5, 30, 30, 30, 30];
  
        // Check if Ethiopian leap year (pagume has 6 days)
        if (ethiopianYear % 4 === 3) {
          ethiopianMonths[10] = 6;
        } else {
          ethiopianMonths[10] = 5;
        }
  
        // Ethiopian new year in Gregorian calendar
        const newYearDay = this.startDayOfEthiopian(ethiopianYear);
  
        // Calculate number of days up to that date
        let until = 0;
        for (let i = 1; i < gMonth; i++) {
          until += gregorianMonths[i];
        }
        until += gDay;
  
        // Update tahissas (december) to match january 1st
        let tahissas;
        if (ethiopianYear % 4 === 0) {
          tahissas = 26;
        } else {
          tahissas = 25;
        }
  
        // Take into account the 1582 change
        if (gYear < 1582) {
          ethiopianMonths[1] = 0;
          ethiopianMonths[2] = tahissas;
        } else if (until <= 277 && gYear === 1582) {
          ethiopianMonths[1] = 0;
          ethiopianMonths[2] = tahissas;
        } else {
          tahissas = newYearDay - 3;
          ethiopianMonths[1] = tahissas;
        }
  
        // Calculate month and date incremently
        let m = 0;
        let ethiopianDate = 0;
        for (m = 1; m < ethiopianMonths.length; m++) {
          if (until <= ethiopianMonths[m]) {
            if (m === 1 || ethiopianMonths[m] === 0) {
              ethiopianDate = until + (30 - tahissas);
            } else {
              ethiopianDate = until;
            }
            break;
          } else {
            until -= ethiopianMonths[m];
          }
        }
  
        // If m > 10, we're already on next Ethiopian year
        if (m > 10) {
          ethiopianYear += 1;
        }
  
        // Ethiopian months ordered according to Gregorian
        const order = [0, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 1, 2, 3, 4];
        const ethiopianMonth = order[m];
  
        return { year: ethiopianYear, month: ethiopianMonth, day: ethiopianDate };
      },
  
      selectDate(day) {

        this.selectedDay = day;
      
      // Get Ethiopian date
      const ethiopianDate = {
        year: this.selectedYear,
        month: this.selectedMonth,
        day: day
      };

      // Convert to Gregorian
      const gregDate = this.ethiopianToGregorian(
        this.selectedYear,
        this.selectedMonth,
        day
      );
      
      // Format Gregorian date
      const gregorianDate = {
        year: gregDate.getFullYear(),
        month: gregDate.getMonth() + 1, // JS months are 0-based
        day: gregDate.getDate()
      };

      this.$emit("date-selected", {
        ethiopianDate,
        gregorianDate
      });
      },
      prevMonth() {
        if (this.selectedMonth === 1) {
          this.selectedMonth = 13;
          this.selectedYear -= 1;
        } else {
          this.selectedMonth -= 1;
        }
        this.selectedDay = null;  
      },
      nextMonth() {
        if (this.selectedMonth === 13) {
          this.selectedMonth = 1;
          this.selectedYear += 1;
        } else {
          this.selectedMonth += 1;
        }
        this.selectedDay = null;  
      },
      prevYear() {
        this.selectedYear -= 1;
        this.selectedDay = null;  
      },
      nextYear() {
        this.selectedYear += 1;
        this.selectedDay = null;  
      }
    }
  };
  </script>
  
  <style >
/* :root {
  --primary-color: #4a86e8;
  --hover-color: #356dca;
  --today-color: #ffe08a;
  --selected-color: #4a86e8;
  --text-color: #2d3748;
  --background-color: #ffffff;
  --border-color: #e2e8f0;
  --header-bg: #f7fafc;
  --weekday-color: #718096;
  --font-stack: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto,
    'Noto Sans Ethiopic', sans-serif;
} */

.calendar-container {
  width: 100%;
  max-width: 400px;
  border: 1px solid var(--border-color);
  border-radius: 12px;
  padding: 16px;
  background: var(--background-color);
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1),
    0 4px 6px -2px rgba(0, 0, 0, 0.05);
  font-family: var(--font-stack);
  overflow: hidden;
}

.calendar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 0;
  margin-bottom: 12px;
  background: var(--header-bg);
  border-radius: 8px;
}

.calendar-header button {
  border: none;
  background: none;
  cursor: pointer;
  padding: 6px 12px;
  border-radius: 6px;
  transition: all 0.2s ease;
  color: var(--text-color);
  font-size: 16px;
}

.calendar-header button:hover {
  background: var(--border-color);
  transform: scale(1.05);
}

.calendar-header button:active {
  transform: scale(0.95);
}

.calendar-header span {
  font-weight: 600;
  color: var(--text-color);
  font-size: 1.1em;
}

.calendar-weekdays {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 4px;
  margin-bottom: 8px;
  color: var(--weekday-color);
  font-weight: 500;
  font-size: 0.9em;
  text-transform: uppercase;
}

.calendar-days {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 4px;
}

.calendar-day {
  position: relative;
  padding: 12px;
  border-radius: 8px;
  cursor: pointer;
  /* transition: all 0.2s ease; */
  color: var(--text-color);
  font-weight: 500;
  background: var(--background-color);
  border: 1px solid transparent;
}

.calendar-day:hover {
    transform: translateY(-1px);
    box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
}

.calendar-day.selected {
  background: var(--selected-color);
  color: rgb(57, 143, 193);
  font-weight: 600;
  border-color: var(--selected-color);
}

.calendar-day.empty {
  visibility: hidden;
}


.calendar-day::after {
  content: '';
  position: absolute;
  bottom: 4px;
  left: 50%;
  transform: translateX(-50%);
  width: 4px;
  height: 4px;
  background: var(--primary-color);
  border-radius: 50%;
  opacity: 0;
}

.calendar-day.today::after {
  opacity: 1;
}

/* Weekend styling 
.calendar-days > .calendar-day:first-child,
.calendar-days > .calendar-day:nth-child(7n) {
  color: var(--primary-color);
}*/

.calendar-footer {
  margin-top: 12px;
  padding: 12px;
  background: var(--header-bg);
  border-radius: 8px;
  font-size: 0.9em;
  color: var(--text-color);
  font-weight: 500;
}

@media (max-width: 380px) {
  .calendar-container {
    border-radius: 0;
    border-left: none;
    border-right: none;
  }
  
  .calendar-header button {
    padding: 4px 8px;
  }
  
  .calendar-day {
    padding: 8px;
    font-size: 0.9em;
  }
}
</style>