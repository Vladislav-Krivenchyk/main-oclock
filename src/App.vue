<template>
  <div id="app">
    <table>
      <thead>
      <tr>
        <th>Day</th>
        <th v-for="hour in hours" :key="hour">{{ hour }}</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="(day, index) in week" :key="index">
        <td @click="toggleRowSelection(day)">all day {{ day }}</td>
        <td
            v-for="hour in hours"
            :key="hour"
            :class="{ active: isSelected(day, hour) }"
            @mousedown="startSelection(day, hour)"
            @mouseover="select(day, hour)"
            @mouseup="endSelection"
            @click="toggleSelection(day, hour)"
        ></td>
      </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      week: ['mo', 'tu', 'we', 'th', 'fr', 'sa', 'su'],
      hours: Array.from({ length: 24 }, (_, i) => i),
      selectedIntervals: [],
      selecting: false,
      startDay: '',
      startHour: '',
      selectedDay: '',
      selectedHour: '',
    };
  },
  computed: {
    jsonOutput() {
      const output = {};
      this.week.forEach((day) => {
        output[day] = this.selectedIntervals
            .filter((interval) => interval.day === day)
            .map((interval) => ({
              bt: interval.startHour * 60,
              et: interval.endHour * 60 + 59,
            }));
      });
      return output;
    },
  },
  methods: {
    toggleRowSelection(day) {
      const isRowSelected = this.isRowSelected(day);

      if (isRowSelected) {
        this.deselectRow(day);
      } else {
        this.selectRow(day);
      }
    },
    isRowSelected(day) {
      return this.selectedIntervals.some((interval) => interval.day === day);
    },
    selectRow(day) {
      for (let hour = 0; hour < 24; hour++) {
        this.selectCell(day, hour);
      }
    },
    toggleSelection(day, hour) {
      const isCellSelected = this.isCellSelected(day, hour);

      if (isCellSelected) {
        this.deselectCell(day, hour);
      } else {
        this.selectCell(day, hour);
      }
    },
    selectCell(day, hour) {
      this.selectedIntervals.push({
        day,
        startHour: hour,
        endHour: hour,
      });
    },
    deselectRow(day) {
      this.selectedIntervals = this.selectedIntervals.filter((interval) => interval.day !== day);
    },
    isCellSelected(day, hour) {
      return this.selectedIntervals.some(
          (interval) => interval.day === day && interval.startHour <= hour && interval.endHour >= hour
      );
    },
    deselectCell(day, hour) {
      this.selectedIntervals = this.selectedIntervals.filter(
          (interval) => !(interval.day === day && interval.startHour <= hour && interval.endHour >= hour)
      );
    },
    isSelected(day, hour) {
      return this.selectedIntervals.some(
          (interval) => interval.day === day && interval.startHour <= hour && interval.endHour >= hour
      );
    },
    startSelection(day, hour) {
      this.selecting = true;
      this.startDay = day;
      this.startHour = hour;
    },
    select(day, hour) {
      if (this.selecting) {
        const start = {
          day: this.startDay,
          startHour: Math.min(this.startHour, hour),
          endHour: Math.max(this.startHour, hour),
        };
        const end = {
          day,
          startHour: Math.min(this.startHour, hour),
          endHour: Math.max(this.startHour, hour),
        };

        this.selectedIntervals.push(start, end);
        this.selectedDay = day;
        this.selectedHour = hour;
      }
    },
    endSelection() {
      this.selecting = false;

      if (!this.selectedIntervals.some((interval) => interval.day === this.selectedDay && interval.startHour === this.selectedHour)) {
        this.selectedIntervals.push({
          day: this.selectedDay,
          startHour: this.selectedHour,
          endHour: this.selectedHour,
        });
      }

      this.selectedDay = '';
      this.selectedHour = '';
    },
  },
  };
</script>

<style>
.active {
  background-color: #333;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th,
td {
  padding: 10px;
  text-align: center;
  border: 1px solid #ccc;
}
td {
  cursor: pointer;
}
</style>
