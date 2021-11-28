<script>
  export let date;
  $: daysPerMonth = [
    31,
    isLeapYear(year) ? 29 : 28,
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
    isLeapYear(year) ? 29 : 28,
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

  let day = 1;
  let month = 0;
  let year = new Date().getFullYear();

  let gregorianDate = new Date().toISOString().split("T")[0];
  $: convertedDate = getIFCDate(gregorianDate);
  $: ifcToGregDate = getGregorianDate(year, month, day);

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
      console.log(total);
      total += m;
      if (doy <= total) {
        return `${gregMonths[index]} ${doy - (total - m)}, ${year}`;
      }
    }
  }

  function incrementMonth() {
    month += 1;
    if (month >= ifcMonths.length) {
      month = 0;
      year += 1;
    }
  }

  function decrementMonth() {
    month -= 1;
    if (month < 0) {
      month = ifcMonths.length - 1;
      year -= 1;
    }
  }

  function toDate(datestring) {
    const [y, m, d] = datestring.split("-");
    return new Date(y, parseInt(m) - 1, d);
  }
</script>

<div id="ifc-to-gregorian">
  <h1>International Fixed Calendar to Gregorian</h1>
  <label for="day-input">Day</label>
  <select id="day-input" bind:value={day}>
    {#each [...Array((isLeapYear(year) && month === 5) || month === 12 ? 29 : 28).keys()].map((x) => x + 1) as d}
      <option value={d}>{d}</option>
    {/each}
  </select>

  <label for="month-input">Month</label>
  <select id="month-input" bind:value={month}>
    {#each ifcMonths as m, index}
      <option value={index}>{m}</option>
    {/each}
  </select>

  <label for="year-input">Day</label>
  <select id="year-input" bind:value={year}>
    {#each [...Array(100).keys()].map((x) => x + 2000) as y}
      <option value={y}>{y}</option>
    {/each}
  </select>

  <div id="chosen-date">
    <p><b>Chosen Date: </b>{ifcMonths[month]} {day}, {year}</p>
  </div>
</div>

<div id="gregorian-to-ifc">
  <h1>Gregorian to IFC</h1>
  <label for="date-input">Gregorian Date</label>
  <input type="date" bind:value={gregorianDate} />
  <p>
    {convertedDate}
  </p>
</div>

{ifcToGregDate}

<div class="ifc-datepicker">
  <div class="datepicker-banner">
    <button class="datepicker-button" on:click={decrementMonth}>&lt;</button>
    <div class="month-year">
      <p>{ifcMonths[month]}</p>
      <select id="year-input" bind:value={year}>
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
    {#each [...Array((isLeapYear(year) && month === 5) || month === 12 ? 29 : 28).keys()].map((x) => x + 1) as d, index}
      <button
        class="datepicker-button"
        class:selected={day === d}
        class:new-row={index % 7 === 0 && d !== 29}
        on:click={() => (day = d)}
      >
        {d}
      </button>
    {/each}
  </div>
</div>

<style>
  .ifc-datepicker {
    border: 1px solid black;
    padding: 1em;
    width: 20rem;
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
