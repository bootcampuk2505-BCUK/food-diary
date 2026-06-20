# food-diary

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Smart Food Diary</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <style>
    body { font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif; margin: 0; padding: 1rem; background:#f7f7f7; }
    h1 { font-size: 1.5rem; text-align: center; margin-top:0; }
    .container { max-width: 900px; margin: 0 auto; }
    .card { background:#fff; padding:1rem; margin-bottom:1rem; border-radius:8px; box-shadow:0 1px 3px rgba(0,0,0,0.08); }
    label { display:block; margin-top:0.5rem; font-size:0.9rem; }
    input, select {
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
    .flex { display:flex; gap:1rem; flex-wrap:wrap; }
    .flex-2 { flex:2 1 300px; }
    .flex-1 { flex:1 1 220px; }
    table { width:100%; border-collapse:collapse; font-size:0.85rem; }
    th, td { padding:0.4rem; border-bottom:1px solid #eee; text-align:left; }
    th { background:#f0f0f0; }
    .entry-actions button { background:none; border:none; color:#c00; cursor:pointer; font-size:0.8rem; padding:0; }
    .metric-row { margin:0.4rem 0; }
    .metric-label { font-size:0.85rem; margin-bottom:0.1rem; display:flex; justify-content:space-between; }
    .bar-bg { width:100%; background:#eee; border-radius:999px; overflow:hidden; height:12px; }
    .bar-fill { height:100%; width:0%; background:#4caf50; transition:width 0.2s linear, background-color 0.2s linear; }
    .small { font-size:0.8rem; color:#666; }
  </style>
</head>
<body>
<div class="container">
  <h1>Smart Food Diary</h1>

  <div class="card flex">
    <div class="flex-2">
      <label for="date">Date</label>
      <input type="date" id="date">

      <label for="foodSelect">Food</label>
      <select id="foodSelect">
        <option value="">Select a food…</option>
      </select>

      <label for="amount">Amount</label>
      <input type="number" id="amount" value="1" min="0.1" step="0.1">

      <span id="unitLabel" class="small"></span>

      <button id="addBtn">Add to Diary</button>
      <button id="clearBtn" class="secondary">Clear This Day</button>
    </div>

    <div class="flex-1">
      <strong>Daily Targets (example)</strong>
      <div class="small">Adjust in code if needed.</div>
      <ul class="small">
        <li>Calories: 2000 kcal</li>
        <li>Protein: 100 g</li>
        <li>Carbs: 250 g</li>
        <li>Fibre: 30 g</li>
        <li>Sugar: 50 g</li>
        <li>Good fats: 70 g</li>
        <li>Bad fats: 20 g</li>
      </ul>
    </div>
  </div>

  <div class="card flex">
    <div class="flex-2">
      <h2 style="font-size:1.1rem;margin-top:0;">Entries</h2>
      <table id="entriesTable">
        <thead>
          <tr>
            <th>Food</th>
            <th>Amount</th>
            <th>Calories</th>
            <th>Protein</th>
            <th>Carbs</th>
            <th>Fibre</th>
            <th>Sugar</th>
            <th>Good fat</th>
            <th>Bad fat</th>
            <th></th>
          </tr>
        </thead>
        <tbody></tbody>
      </table>
    </div>

    <div class="flex-1">
      <h2 style="font-size:1.1rem;margin-top:0;">Daily Progress</h2>
      <div id="summary" class="small" style="margin-bottom:0.5rem;"></div>
      <div id="metricsBars"></div>
    </div>
  </div>
</div>

<script>
  // ---------------- FOOD DATABASE ----------------
  const FOOD_DB = [
    {
      name: "Apple (medium)",
      unit: "item",
      calories: 95,
      protein: 0.5,
      carbs: 25,
      fibre: 4.4,
      sugar: 19,
      goodFat: 0.2,
      badFat: 0
    },
    {
      name: "Banana (medium)",
      unit: "item",
      calories: 105,
      protein: 1.3,
      carbs: 27,
      fibre: 3.1,
      sugar: 14,
      goodFat: 0.4,
      badFat: 0
    },
    {
      name: "Chicken breast (100g, cooked)",
      unit: "portion (100 g)",
      calories: 165,
      protein: 31,
      carbs: 0,
      fibre: 0,
      sugar: 0,
      goodFat: 3.6,
      badFat: 1
    },
    {
      name: "Olive oil (1 tbsp)",
      unit: "tbsp",
      calories: 119,
      protein: 0,
      carbs: 0,
      fibre: 0,
      sugar: 0,
      goodFat: 13.5,
      badFat: 2
    },
    {
      name: "White rice (100g cooked)",
      unit: "portion (100 g)",
      calories: 130,
      protein: 2.7,
      carbs: 28,
      fibre: 0.4,
      sugar: 0.1,
      goodFat: 0.3,
      badFat: 0.1
    },
    {
      name: "Broccoli (100g, cooked)",
      unit: "portion (100 g)",
      calories: 55,
      protein: 3.7,
      carbs: 11,
      fibre: 3.8,
      sugar: 2.2,
      goodFat: 0.6,
      badFat: 0
    },
    {
      name: "Whole egg (large)",
      unit: "item",
      calories: 72,
      protein: 6.3,
      carbs: 0.4,
      fibre: 0,
      sugar: 0.2,
      goodFat: 3,
      badFat: 1.6
    },
    {
      name: "Peanut butter (1 tbsp)",
      unit: "tbsp",
      calories: 94,
      protein: 4,
      carbs: 3.2,
      fibre: 1,
      sugar: 1.5,
      goodFat: 7.8,
      badFat: 1.6
    },
    {
      name: "Oats (40g dry)",
      unit: "portion (40 g)",
      calories: 150,
      protein: 5,
      carbs: 27,
      fibre: 4,
      sugar: 1,
      goodFat: 3,
      badFat: 0.5
    }
  ];

  // ---------------- DAILY TARGETS ----------------
  const DAILY_TARGETS = {
    calories: 2000,
    protein: 100,
    carbs: 250,
    fibre: 30,
    sugar: 50,
    goodFat: 70,
    badFat: 20
  };

  const METRICS_CONFIG = [
    { key: "calories", label: "Calories (kcal)" },
    { key: "protein",  label: "Protein (g)" },
    { key: "carbs",    label: "Carbs (g)" },
    { key: "fibre",    label: "Fibre (g)" },
    { key: "sugar",    label: "Sugar (g)" },
    { key: "goodFat",  label: "Good fats (g)" },
    { key: "badFat",   label: "Bad fats (g)" }
  ];

  // Run after DOM is ready
  window.addEventListener("DOMContentLoaded", () => {
    const dateInput   = document.getElementById("date");
    const foodSelect  = document.getElementById("foodSelect");
    const amountInput = document.getElementById("amount");
    const unitLabel   = document.getElementById("unitLabel");
    const addBtn      = document.getElementById("addBtn");
    const clearBtn    = document.getElementById("clearBtn");
    const entriesTBody= document.querySelector("#entriesTable tbody");
    const summaryDiv  = document.getElementById("summary");
    const metricsBars = document.getElementById("metricsBars");

    const todayStr = new Date().toISOString().split("T")[0];
    dateInput.value = todayStr;

    // Populate dropdown
    FOOD_DB.forEach((f, idx) => {
      const opt = document.createElement("option");
      opt.value = idx;
      opt.textContent = f.name;
      foodSelect.appendChild(opt);
    });

    function getSelectedFood() {
      const idx = parseInt(foodSelect.value, 10);
      if (isNaN(idx)) return null;
      return FOOD_DB[idx];
    }

    function updateUnitLabel() {
      const food = getSelectedFood();
      unitLabel.textContent = food ? `Per ${food.unit}. Amount = how many ${food.unit}s.` : "";
    }
    updateUnitLabel();
    foodSelect.addEventListener("change", updateUnitLabel);

    // Local storage helpers
    function loadAllData() {
      return JSON.parse(localStorage.getItem("smartFoodDiary") || "{}");
    }
    function saveAllData(data) {
      localStorage.setItem("smartFoodDiary", JSON.stringify(data));
    }
    function getCurrentDate() {
      return dateInput.value || todayStr;
    }
    function loadEntriesForDate(dateStr) {
      const all = loadAllData();
      return all[dateStr] || [];
    }
    function saveEntriesForDate(dateStr, entries) {
      const all = loadAllData();
      all[dateStr] = entries;
      saveAllData(all);
    }

    function renderBars(totals) {
      metricsBars.innerHTML = "";
      METRICS_CONFIG.forEach(m => {
        const total = totals[m.key] || 0;
        const target = DAILY_TARGETS[m.key];
        const pct = target > 0 ? (total / target) * 100 : 0;

        const row = document.createElement("div");
        row.className = "metric-row";

        const label = document.createElement("div");
        label.className = "metric-label";
        label.innerHTML = `
          <span>${m.label}</span>
          <span>${total.toFixed(1)} / ${target}${m.key === "calories" ? " kcal" : " g"} (${pct.toFixed(0)}%)</span>
        `;

        const barBg = document.createElement("div");
        barBg.className = "bar-bg";

        const barFill = document.createElement("div");
        barFill.className = "bar-fill";
        const width = Math.min(pct, 130);
        barFill.style.width = width + "%";

        let color = "#4caf50";
        if (pct >= 90 && pct <= 100) color = "#ff9800";
        if (pct > 100) color = "#f44336";
        barFill.style.backgroundColor = color;

        barBg.appendChild(barFill);
        row.appendChild(label);
        row.appendChild(barBg);
        metricsBars.appendChild(row);
      });
    }

    function renderEntries() {
      const dateStr = getCurrentDate();
      const entries = loadEntriesForDate(dateStr);
      entriesTBody.innerHTML = "";

      let totals = {
        calories: 0,
        protein: 0,
        carbs: 0,
        fibre: 0,
        sugar: 0,
        goodFat: 0,
        badFat: 0
      };

      entries.forEach((e, i) => {
        const tr = document.createElement("tr");
        function td(text) {
          const c = document.createElement("td");
          c.textContent = text;
          return c;
        }
        tr.appendChild(td(e.name));
        tr.appendChild(td(e.amount + " " + e.unit));
        tr.appendChild(td(e.calories.toFixed(0)));
        tr.appendChild(td(e.protein.toFixed(1)));
        tr.appendChild(td(e.carbs.toFixed(1)));
        tr.appendChild(td(e.fibre.toFixed(1)));
        tr.appendChild(td(e.sugar.toFixed(1)));
        tr.appendChild(td(e.goodFat.toFixed(1)));
        tr.appendChild(td(e.badFat.toFixed(1)));

        const actTd = document.createElement("td");
        actTd.className = "entry-actions";
        const delBtn = document.createElement("button");
        delBtn.textContent = "✕";
        delBtn.title = "Delete";
        delBtn.addEventListener("click", () => {
          const ents = loadEntriesForDate(dateStr);
          ents.splice(i, 1);
          saveEntriesForDate(dateStr, ents);
          renderEntries();
        });
        actTd.appendChild(delBtn);
        tr.appendChild(actTd);

        entriesTBody.appendChild(tr);

        totals.calories += e.calories;
        totals.protein  += e.protein;
        totals.carbs    += e.carbs;
        totals.fibre    += e.fibre;
        totals.sugar    += e.sugar;
        totals.goodFat  += e.goodFat;
        totals.badFat   += e.badFat;
      });

      summaryDiv.textContent = entries.length
        ? `Items: ${entries.length} · Calories: ${totals.calories.toFixed(0)} kcal`
        : "No entries for this day.";

      renderBars(totals);
    }

    addBtn.addEventListener("click", () => {
      const dateStr = getCurrentDate();
      const food = getSelectedFood();
      const amt = parseFloat(amountInput.value) || 0;
      if (!food || amt <= 0) return;

      const entries = loadEntriesForDate(dateStr);
      const scale = v => v * amt;

      const entry = {
        name: food.name,
        unit: food.unit,
        amount: amt,
        calories: scale(food.calories),
        protein: scale(food.protein),
        carbs: scale(food.carbs),
        fibre: scale(food.fibre),
        sugar: scale(food.sugar),
        goodFat: scale(food.goodFat),
        badFat: scale(food.badFat)
      };

      entries.push(entry);
      saveEntriesForDate(dateStr, entries);
      renderEntries();
    });

    clearBtn.addEventListener("click", () => {
      const dateStr = getCurrentDate();
      if (!confirm("Clear all entries for this day?")) return;
      const all = loadAllData();
      delete all[dateStr];
      saveAllData(all);
      renderEntries();
    });

    dateInput.addEventListener("change", renderEntries);

    renderEntries();
  });
</script>
</body>
</html>
