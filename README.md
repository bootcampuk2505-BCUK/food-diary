# food-diary

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Food Diary</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <style>
    body { font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif; margin: 0; padding: 1rem; background:#f7f7f7; }
    h1 { font-size: 1.6rem; text-align: center; margin-top:0; }
    .card { background:#fff; padding:1rem; margin-bottom:1rem; border-radius:8px; box-shadow:0 1px 3px rgba(0,0,0,0.08); max-width:600px; margin-left:auto; margin-right:auto; }
    label { display:block; margin-top:0.5rem; font-size:0.9rem; }
    input, textarea, select {
      width:100%; padding:0.4rem; margin-top:0.2rem;
      border:1px solid #ccc; border-radius:4px; box-sizing:border-box;
      font-size:0.9rem;
    }
    button {
      padding:0.5rem 1rem; border:none; border-radius:4px;
      background:#007bff; color:#fff; font-size:0.9rem; cursor:pointer;
      margin-top:0.8rem;
    }
    button.secondary { background:#6c757d; margin-left:0.5rem; }
    .entries { max-height:60vh; overflow-y:auto; }
    .entry { border-bottom:1px solid #eee; padding:0.5rem 0; }
    .entry:last-child { border-bottom:none; }
    .entry-header { display:flex; justify-content:space-between; font-size:0.85rem; color:#555; }
    .entry-meal { font-weight:bold; }
    .entry-notes { font-size:0.9rem; margin-top:0.2rem; white-space:pre-wrap; }
    .entry-meta { font-size:0.8rem; color:#777; margin-top:0.2rem; display:flex; justify-content:space-between; align-items:center; gap:0.5rem; flex-wrap:wrap; }
    .delete-btn {
      background:transparent; border:none; color:#c00; cursor:pointer; font-size:0.8rem;
      padding:0;
    }
  </style>
</head>
<body>
  <h1>Food Diary</h1>

  <div class="card">
    <label for="date">Date</label>
    <input type="date" id="date">

    <label for="meal">Meal</label>
    <select id="meal">
      <option>Breakfast</option>
      <option>Snack (AM)</option>
      <option>Lunch</option>
      <option>Snack (PM)</option>
      <option>Dinner</option>
      <option>Other</option>
    </select>

    <label for="food">What did you eat?</label>
    <textarea id="food" rows="2" placeholder="e.g. 2 eggs, toast, coffee"></textarea>

    <label for="calories">Calories (optional)</label>
    <input type="number" id="calories" min="0" placeholder="e.g. 450">

    <label for="notes">Notes (optional)</label>
    <textarea id="notes" rows="2" placeholder="e.g. Felt full, craving something sweet later"></textarea>

    <button id="add-btn">Add Entry</button>
    <button id="clear-day-btn" class="secondary">Clear This Day</button>
  </div>

  <div class="card">
    <h2 style="font-size:1.1rem;margin-top:0;">Entries</h2>
    <div id="summary" style="font-size:0.9rem;margin-bottom:0.5rem;color:#555;"></div>
    <div class="entries" id="entries"></div>
  </div>

  <script>
    const dateInput    = document.getElementById('date');
    const mealInput    = document.getElementById('meal');
    const foodInput    = document.getElementById('food');
    const calInput     = document.getElementById('calories');
    const notesInput   = document.getElementById('notes');
    const addBtn       = document.getElementById('add-btn');
    const clearDayBtn  = document.getElementById('clear-day-btn');
    const entriesDiv   = document.getElementById('entries');
    const summaryDiv   = document.getElementById('summary');

    // Default to today
    const today = new Date().toISOString().split('T')[0];
    dateInput.value = today;

    function loadAllData() {
      return JSON.parse(localStorage.getItem('foodDiary') || '{}');
    }

    function loadEntries() {
      const allData = loadAllData();
      const day = dateInput.value;
      return allData[day] || [];
    }

    function saveEntries(entries) {
      const allData = loadAllData();
      allData[dateInput.value] = entries;
      localStorage.setItem('foodDiary', JSON.stringify(allData));
    }

    function renderEntries() {
      const entries = loadEntries();
      entriesDiv.innerHTML = '';
      let totalCal = 0;

      entries.forEach((e, idx) => {
        const wrap = document.createElement('div');
        wrap.className = 'entry';

        if (e.calories) totalCal += Number(e.calories) || 0;

        wrap.innerHTML = `
          <div class="entry-header">
            <div><span class="entry-meal">${e.meal}</span></div>
            <div>${e.time || ''}</div>
          </div>
          <div class="entry-notes">${e.food}</div>
          <div class="entry-meta">
            <div>
              ${e.calories ? `Calories: ${e.calories}` : ''}
              ${e.notes ? `<br>Notes: ${e.notes}` : ''}
            </div>
            <button class="delete-btn" data-idx="${idx}">Delete</button>
          </div>
        `;
        entriesDiv.appendChild(wrap);
      });

      summaryDiv.textContent = entries.length
        ? `Entries: ${entries.length}` + (totalCal ? ` · Total calories: ${totalCal}` : '')
        : 'No entries yet for this day.';

      document.querySelectorAll('.delete-btn').forEach(btn => {
        btn.addEventListener('click', () => {
          const idx = Number(btn.getAttribute('data-idx'));
          const ents = loadEntries();
          ents.splice(idx, 1);
          saveEntries(ents);
          renderEntries();
        });
      });
    }

    addBtn.addEventListener('click', () => {
      const food = foodInput.value.trim();
      if (!food) return;

      const entries = loadEntries();
      const now = new Date();
      const timeStr = now.toTimeString().substring(0,5); // HH:MM

      entries.push({
        meal: mealInput.value,
        food: food,
        calories: calInput.value.trim(),
        notes: notesInput.value.trim(),
        time: timeStr
      });

      saveEntries(entries);
      foodInput.value = '';
      calInput.value = '';
      notesInput.value = '';
      renderEntries();
    });

    clearDayBtn.addEventListener('click', () => {
      if (!confirm('Clear all entries for this day?')) return;
      const allData = loadAllData();
      delete allData[dateInput.value];
      localStorage.setItem('foodDiary', JSON.stringify(allData));
      renderEntries();
    });

    dateInput.addEventListener('change', renderEntries);

    // Initial render
    renderEntries();
  </script>
</body>
</html>
