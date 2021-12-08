<script>
  // TODO: Improve efficiency of comparing IFC date to current date
  export let date;
  let selectedDay = 1;
  let selectedMonth = 0;
  let selectedYear = date.getFullYear();
  let activeMonth = selectedMonth;
  let activeYear = selectedYear;
  $: daysPerMonth = [
    31,
    isLeapYear(selectedYear) ? 29 : 28,
    31,
    30,
    31,
    30,
    31,
    31,
    30,
    31,
    30,
    31,
  ];
  $: ifcDaysPerMonth = [
    28,
    28,
    28,
    28,
    28,
    isLeapYear(selectedYear) ? 29 : 28,
    28,
    28,
    28,
    28,
    28,
    28,
    29,
  ];
  const ifcMonths = [
    "Jan",
    "Feb",
    "Mar",
    "Apr",
    "May",
    "Jun",
    "Sol",
    "Jul",
    "Aug",
    "Sep",
    "Oct",
    "Nov",
    "Dec",
  ];

  const daysOfWeek = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];
  $: ifcDate = `${ifcMonths[selectedMonth]} ${selectedDay}, ${selectedYear}`;

  function dayOfYear(d) {
    const currentMonth = d.getMonth();
    let doy =
      daysPerMonth
        .slice(0, currentMonth)
        .reduce((prev, current) => prev + current, 0) + d.getDate();
    if (currentMonth > 1 && isLeapYear(d.getFullYear())) {
      doy += 1;
    }
    return doy;
  }

  function isLeapYear(year) {
    return (
      (year % 4 === 0 && year % 100 !== 0) ||
      (year % 4 === 0 && year % 100 === 0 && year % 400 === 0)
    );
  }

  function getIFCDate(datestring) {
    const selectedDate = toDate(datestring);
    const doy = dayOfYear(selectedDate);
    let total = 0;
    for (const [index, m] of ifcDaysPerMonth.entries()) {
      total += m;
      if (doy <= total) {
        const ifcDateString = `${ifcMonths[index]} ${
          doy - (total - m)
        }, ${selectedDate.getFullYear()}`;
        return ifcDateString;
      }
    }
  }

  function getGregorianDate(y, m, d) {
    // Get Day of Year
    let doy = m * 28 + d;
    if (m > 5 && isLeapYear(y)) {
      doy += 1;
    }

    // Find currentMonth
    let gregMonths = [...ifcMonths];
    gregMonths.slice(6, 1);

    let total = 0;

    for (const [index, m] of daysPerMonth.entries()) {
      total += m;
      if (doy <= total) {
        return new Date(selectedYear, index, doy - (total - m));
        // return `${gregMonths[index]} ${doy - (total - m)}, ${selectedYear}`;
      }
    }
  }

  function incrementMonth() {
    activeMonth += 1;
    if (activeMonth >= ifcMonths.length) {
      activeMonth = 0;
      activeYear += 1;
    }
  }

  function decrementMonth() {
    activeMonth -= 1;
    if (activeMonth < 0) {
      activeMonth = ifcMonths.length - 1;
      activeYear -= 1;
    }
  }

  function toDate(datestring) {
    const [y, m, d] = datestring.split("-");
    return new Date(y, parseInt(m) - 1, d);
  }

  function handleDayChange(activeDay) {
    selectedDay = activeDay;
    selectedMonth = activeMonth;
    selectedYear = activeYear;
    date = getGregorianDate(selectedYear, selectedMonth, selectedDay);
  }
</script>

<div class="datepicker-container">
  <input id="ifc-date-input" value={ifcDate} />
  <div class="ifc-datepicker">
    <div class="datepicker-banner">
      <button class="datepicker-button" on:click={decrementMonth}>&lt;</button>
      <div class="month-year">
        <p>{ifcMonths[activeMonth]}</p>
        <select id="year-input" bind:value={activeYear}>
          {#each [...Array(100).keys()].map((x) => x + 2000) as y}
            <option value={y}>{y}</option>
          {/each}
        </select>
      </div>
      <button class="datepicker-button" on:click={incrementMonth}>&gt;</button>
    </div>

    <div class="calendar">
      {#each daysOfWeek as dow, index}
        <p class:new-row={index % 7 === 0}>{dow}</p>
      {/each}
      {#each [...Array((isLeapYear(activeYear) && activeMonth === 5) || activeMonth === 12 ? 29 : 28).keys()].map((x) => x + 1) as d, index}
        <button
          class="datepicker-button"
          class:selected={activeYear === selectedYear &&
            activeMonth === selectedMonth &&
            selectedDay === d}
          class:currentdate={dayOfYear(
            getGregorianDate(activeYear, activeMonth, d)
          ) === dayOfYear(new Date())}
          class:new-row={index % 7 === 0 && d !== 29}
          on:click={() => handleDayChange(d)}
        >
          {d}
        </button>
      {/each}
    </div>
  </div>
</div>

<style>
  .ifc-datepicker {
    border: 1px solid black;
    padding: 1em;
    width: 20rem;
    display: none;
  }

  .datepicker-container:focus-within .ifc-datepicker {
    display: block;
  }

  .datepicker-banner {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
  }

  .month-year {
    display: flex;
  }

  .calendar {
    display: grid;
    grid-template-columns: repeat(8, 1fr);
  }

  .new-row {
    grid-column-start: 1;
  }

  .datepicker-button {
    color: black;
    background-color: white;
    border-radius: 50%;
    border: none;
    aspect-ratio: 1/1;
    width: 2.5em;
    height: 2.5em;
  }

  .datepicker-button.selected {
    color: white;
    background-color: #6200ee;
    border-radius: 50%;
    border: none;
  }

  .datepicker-button.currentdate {
    border: 1px solid black;
  }

  .datepicker-button:not(.selected):hover {
    border: 1px solid #6200ee;
    cursor: pointer;
  }

  .datepicker-button:not(.selcted):active:hover {
    border: 1px solid #03dac6;
  }

  .datepicker-button:active {
    background-color: #03dac6;
    border: 1px solid #03dac6;
  }
</style>
