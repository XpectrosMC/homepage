

# Xpectros

<style>
      body {
        margin: 0;
        padding: 0;
        background-color: #121212;
        color: #e0e0e0;
        font-family: sans-serif;
      }

      * {
        box-sizing: border-box;
      }

      .calendar {
        display: flex;
        flex-direction: column;
        gap: 1rem;
        padding: 1rem;
        width: 100%;
        box-sizing: border-box;
        --accent-color: #663cfc;
      }

      .day {
        display: flex;
        padding: 0.75rem;
        border-radius: 8px;
        background-color: #1e1e1e;
        box-shadow: 0 0 4px rgba(0, 0, 0, 0.5);
        width: 100%;
      }

      .date {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        margin-right: 1rem;
        width: 50px;
        color: var(--accent-color);
      }

      .number {
        font-size: 1.5rem;
        font-weight: bold;
      }

      .month {
        font-size: 0.75rem;
        color: #cccccc;
        margin-top: 0.2rem;
      }

      .events {
        display: flex;
        flex-direction: column;
        gap: 0.5rem;
        flex-grow: 1;
      }

      .event {
        background-color: #2c2c2c;
        padding: 0.5rem 0.75rem;
        border-radius: 6px;
        color: #ffffff;
        border-left: 4px solid var(--accent-color);
      }
</style>
<div class="templates" style="display:none;">
  <div class="event-template event">[name]</div>
  <div class="day-template day">
    <div class="date">
      <div class="number">[day]</div>
      <div class="month">[month]</div>
    </div>
    <div class="events">
    </div>
  </div>
</div>
<div class="calendar" id="calendar">
  <!-- Calendar days will be populated here -->
</div>
<script>
  const populateDays = days => {
    const calendarEl = document.getElementById('calendar')
    const dayTemplate = document.querySelector('.day-template')
    const eventTemplate = document.querySelector('.event-template')

    Object.keys(days).sort().forEach(dayKey => {
      const dayEl = dayTemplate.cloneNode(true)
      dayEl.classList.remove('day-template')
      dayEl.querySelector('.number').textContent = dayKey.slice(8, 10)
      dayEl.querySelector('.month').textContent = new Date(dayKey).toLocaleString('default', { month: 'short' })

      const eventsContainer = dayEl.querySelector('.events')
      days[dayKey].forEach(event => {
        const eventEl = eventTemplate.cloneNode(true)
        eventEl.classList.remove('event-template')
        let eventText = event.name
        if (!event.allDay) {
          eventText += ` (${event.startTime} - ${event.endTime})`
        }
        eventEl.textContent = eventText
        eventsContainer.appendChild(eventEl)
      })

      calendarEl.appendChild(dayEl)
    })
  }
  const populateCalendar = element => {
    fetch('https://n8n.floresbenavides.com/webhook/calendario')
      .then(response => response.json())
      .then(data => {
        // data = [
        // { start: '2025-01-01', name: 'event1' end: '2025-01-02'},
        // { start: '2025-01-01T19:00:00-06:00', name: 'event2' end: '2025-01-01T22:00:00-06:00'}
        // ]
        data.map(event => {
          // if event.start contains time, it means its sigle day
          event.allDay = event.start.length <= 10
          // if event.allDay, substract 1 day from end date
          if (event.allDay) {
            const endDate = new Date(event.end)
            endDate.setDate(endDate.getDate() - 1)
            event.end = endDate.toISOString().slice(0, 10)
          } else {
            // for single day events set start and end time
            const startDateTime = new Date(event.start)
            const endDateTime = new Date(event.end)
            event.startTime = startDateTime.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
            event.endTime = endDateTime.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
          }
          return event
        })
        const days = {}
        data.forEach(event => {
          const dayKey = event.start.slice(0, 10)
          if (!days[dayKey]) {
            days[dayKey] = []
          }
          if (!event.allDay) days[dayKey].push(event)
          else {
            const startDate = new Date(event.start)
            const endDate = new Date(event.end)
            const timeDiff = endDate.getTime() - startDate.getTime()
            const dayCount = Math.ceil(timeDiff / (1000 * 3600 * 24)) + 1
            let currentDay = 1
            for (let d = new Date(startDate); d <= endDate; d.setDate(d.getDate() + 1)) {
              const evt = Object.assign({}, event)
              const multiDayKey = d.toISOString().slice(0, 10)
              if (!days[multiDayKey]) {
                days[multiDayKey] = []
              }
              // if only one day, keep the name
              evt.name = dayCount === 1 ? evt.name : `${evt.name} (día ${currentDay}/${dayCount})`
              days[multiDayKey].push(evt)
              currentDay++
            }
          }
        })
        populateDays(days)
      })
      .catch(error => {
        element.innerHTML = 'Error loading calendar data.'
        console.error('Error fetching calendar data:', error)
      })
  }
  document.addEventListener('DOMContentLoaded', function() {
    var calendarEl = document.getElementById('calendar')
    populateCalendar(calendarEl)
  })
</script>
