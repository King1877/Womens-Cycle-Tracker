<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>WOMEN'S CYCLE TRACKER</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body { 
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Arial, sans-serif;
      background: linear-gradient(to bottom, #fff5f8, #ffffff);
      min-height: 100vh;
      padding: 20px;
    }
    .container { max-width: 900px; margin: 0 auto; }
    header { text-align: center; margin-bottom: 30px; }
    h1 { color: #e91e63; font-size: 2.5rem; margin-bottom: 10px; }
    .subtitle { color: #666; font-size: 1.1rem; }
    
    .tabs { display: flex; gap: 10px; border-bottom: 2px solid #f0f0f0; margin-bottom: 20px; flex-wrap: wrap; }
    .tab { 
      padding: 12px 24px; border: none; background: transparent; 
      cursor: pointer; font-weight: bold; color: #666;
      border-radius: 8px 8px 0 0; transition: all 0.3s;
    }
    .tab.active { background: #e91e63; color: white; }
    .tab:hover { background: #f8bbd0; }
    
    .card { 
      background: white; padding: 25px; border-radius: 12px; 
      box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin-bottom: 20px;
    }
    
    label { display: block; margin-bottom: 15px; }
    label strong { display: block; margin-bottom: 5px; color: #333; }
    input[type="date"], input[type="number"] {
      width: 100%; padding: 10px; border: 1px solid #ddd;
      border-radius: 6px; font-size: 1rem;
    }
    input[type="range"] { width: 100%; }
    textarea {
      width: 100%; min-height: 80px; padding: 10px;
      border: 1px solid #ddd; border-radius: 6px;
      resize: vertical; font-family: inherit;
    }
    
    .calendar-nav { 
      display: flex; justify-content: space-between; 
      align-items: center; margin-bottom: 20px;
    }
    .btn {
      padding: 10px 20px; border: 1px solid #ddd;
      border-radius: 6px; background: white;
      cursor: pointer; font-weight: bold;
    }
    .btn:hover { background: #f5f5f5; }
    .btn-save {
      background: #e91e63; color: white; border: none;
      margin-top: 10px;
    }
    .btn-export {
      background: #4caf50; color: white; border: none;
      margin-right: 10px;
    }
    .btn-import {
      background: #2196f3; color: white; border: none;
    }
    
    .legend { display: flex; gap: 20px; margin-bottom: 15px; flex-wrap: wrap; }
    .legend-item { display: flex; align-items: center; gap: 8px; font-size: 0.9rem; }
    .legend-box { width: 20px; height: 20px; border-radius: 4px; }
    
    .calendar-grid { 
      display: grid; grid-template-columns: repeat(7, 1fr); 
      gap: 5px; margin-top: 10px;
    }
    .weekday { 
      text-align: center; font-weight: bold; 
      color: #666; padding: 10px; font-size: 0.9rem;
    }
    .day { 
      background: #f9f9f9; border-radius: 8px; 
      padding: 10px; min-height: 90px; cursor: pointer;
      border: 1px solid #eee; transition: transform 0.2s;
      position: relative;
    }
    .day:hover { transform: scale(1.05); }
    .day.today { border: 2px solid #e91e63; }
    .day.period { background: #ffcdd2; }
    .day.fertile { background: #fff9c4; }
    .day.ovulation { background: #b3e5fc; }
    .day-num { font-weight: bold; margin-bottom: 5px; }
    .day-tag { 
      font-size: 0.7rem; background: #e91e63; 
      color: white; padding: 2px 6px; 
      border-radius: 3px; display: inline-block; margin-bottom: 3px;
    }
    .day-note { 
      font-size: 0.75rem; color: #666; 
      overflow: hidden; text-overflow: ellipsis;
      white-space: nowrap; margin-top: 5px;
    }
    
    .predictions { 
      background: #f9f9f9; padding: 15px; 
      border-radius: 8px; margin-top: 20px;
    }
    .predictions h3 { color: #e91e63; margin-bottom: 10px; }
    
    .symptom-entry { 
      background: #f9f9f9; padding: 15px; 
      border-radius: 8px; margin-bottom: 15px;
    }
    .symptom-date { color: #e91e63; font-weight: bold; margin-bottom: 10px; }
    
    .disclaimer {
      background: #fff3e0; padding: 15px; 
      border-radius: 8px; text-align: center;
      font-size: 0.9rem; color: #666; margin-top: 30px;
    }
    
    .hidden { display: none; }
    
    .note-editor {
      margin-top: 20px; padding: 15px; 
      background: #f9f9f9; border-radius: 8px;
    }

    .data-controls {
      margin-top: 20px;
      padding: 15px;
      background: #f0f0f0;
      border-radius: 8px;
    }

    .data-controls h3 {
      margin-bottom: 10px;
      color: #333;
    }

    .cycle-info {
      background: #e3f2fd;
      padding: 15px;
      border-radius: 8px;
      margin-bottom: 15px;
    }

    .cycle-info h4 {
      color: #1976d2;
      margin-bottom: 8px;
    }

    @media (max-width: 768px) {
      h1 { font-size: 2rem; }
      .tabs { gap: 5px; }
      .tab { padding: 10px 15px; font-size: 0.9rem; }
      .day { min-height: 70px; padding: 5px; }
    }
  </style>
</head>
<body>
  <div class="container">
    <header>
      <h1>🌸 Women's Cycle Tracker</h1>
      <p class="subtitle">Track your cycle, symptoms, and wellness</p>
    </header>

    <div class="tabs">
      <button class="tab active" onclick="showTab('calendar')">📅 Calendar</button>
      <button class="tab" onclick="showTab('symptoms')">📊 Symptoms</button>
      <button class="tab" onclick="showTab('settings')">⚙️ Settings</button>
    </div>

    <div id="settings-tab" class="card hidden">
      <h2 style="color: #e91e63; margin-bottom: 20px;">Cycle Settings</h2>
      
      <label>
        <strong>Last Period Start Date:</strong>
        <input type="date" id="lastPeriodDate" onchange="saveSettings()">
      </label>

      <label>
        <strong>Average Cycle Length (days):</strong>
        <input type="number" id="cycleLength" value="28" min="20" max="40" onchange="saveSettings()">
        <small style="color: #666;">Typical range: 21-35 days</small>
      </label>

      <label>
        <strong>Average Period Length (days):</strong>
        <input type="number" id="periodLength" value="5" min="1" max="10" onchange="saveSettings()">
        <small style="color: #666;">Typical range: 3-7 days</small>
      </label>

      <div class="cycle-info" id="cycleInfo"></div>

      <div id="predictions" class="predictions"></div>

      <div class="data-controls">
        <h3>📁 Data Management</h3>
        <button class="btn btn-export" onclick="exportData()">⬇️ Export Data</button>
        <button class="btn btn-import" onclick="document.getElementById('importFile').click()">⬆️ Import Data</button>
        <input type="file" id="importFile" accept=".json" style="display: none;" onchange="importData(event)">
        <p style="margin-top: 10px; font-size: 0.85rem; color: #666;">
          Export your data to backup, or import previously saved data.
        </p>
      </div>
    </div>

    <div id="calendar-tab" class="card">
      <div class="calendar-nav">
        <button class="btn" onclick="changeMonth(-1)">← Previous</button>
        <h2 id="monthName" style="color: #e91e63;"></h2>
        <button class="btn" onclick="changeMonth(1)">Next →</button>
      </div>

      <div style="margin-bottom: 15px;">
        <div class="legend">
          <div class="legend-item">
            <div class="legend-box" style="background: #ffcdd2;"></div>
            Period
          </div>
          <div class="legend-item">
            <div class="legend-box" style="background: #fff9c4;"></div>
            Fertile Window
          </div>
          <div class="legend-item">
            <div class="legend-box" style="background: #b3e5fc;"></div>
            Ovulation
          </div>
        </div>
        <p style="margin-top: 10px; font-size: 0.9rem; color: #666;">
          💡 <strong>Tip:</strong> Right-click (or long-press on mobile) any day to mark/unmark period
        </p>
      </div>

      <div class="calendar-grid">
        <div class="weekday">Sun</div>
        <div class="weekday">Mon</div>
        <div class="weekday">Tue</div>
        <div class="weekday">Wed</div>
        <div class="weekday">Thu</div>
        <div class="weekday">Fri</div>
        <div class="weekday">Sat</div>
      </div>

      <div id="calendarDays" class="calendar-grid"></div>

      <div id="noteEditor" class="note-editor hidden">
        <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 10px;">
          <h3 id="noteDate" style="margin: 0;"></h3>
          <button class="btn" onclick="togglePeriodDay(selectedDate)" style="background: #ff4081; color: white; border: none; padding: 5px 10px; font-size: 0.85rem;">
            <span id="periodToggleText">🩸 Mark as Period</span>
          </button>
        </div>
        <textarea id="noteText" placeholder="Add notes for this day..."></textarea>
        <button class="btn btn-save" onclick="saveNote()">Save Note</button>
      </div>
    </div>

    <div id="symptoms-tab" class="card hidden">
      <h2 style="color: #e91e63; margin-bottom: 20px;">Symptom Tracker</h2>

      <label>
        <strong>Select Date to Track:</strong>
        <input type="date" id="symptomDate" onchange="addSymptomEntry()">
      </label>

      <div id="symptomEntries"></div>
    </div>

    <div class="disclaimer">
      <strong>⚠️ Important:</strong> This app provides estimates based on average cycles. 
      It is not a substitute for medical advice. Consult a healthcare provider for medical concerns or family planning.
    </div>
  </div>

  <script>
    var monthOffset = 0;
    var selectedDate = '';
    var settings = { lastPeriodDate: '', cycleLength: 28, periodLength: 5 };
    var notes = {};
    var symptoms = {};
    var manualPeriodDays = {};

    function loadData() {
      try {
        var saved = localStorage.getItem('cycleTrackerData');
        if (saved) {
          var data = JSON.parse(saved);
          settings = data.settings || settings;
          notes = data.notes || {};
          symptoms = data.symptoms || {};
          manualPeriodDays = data.manualPeriodDays || {};
          
          document.getElementById('lastPeriodDate').value = settings.lastPeriodDate;
          document.getElementById('cycleLength').value = settings.cycleLength;
          document.getElementById('periodLength').value = settings.periodLength;
        }
      } catch (e) {
        console.log('Error loading data:', e);
      }
    }

    function saveData() {
      try {
        localStorage.setItem('cycleTrackerData', JSON.stringify({ 
          settings: settings, 
          notes: notes, 
          symptoms: symptoms,
          manualPeriodDays: manualPeriodDays
        }));
      } catch (e) {
        console.log('Error saving data:', e);
      }
    }

    function saveSettings() {
      settings.lastPeriodDate = document.getElementById('lastPeriodDate').value;
      settings.cycleLength = parseInt(document.getElementById('cycleLength').value);
      settings.periodLength = parseInt(document.getElementById('periodLength').value);
      saveData();
      updatePredictions();
      updateCycleInfo();
      renderCalendar();
    }

    function addDays(date, days) {
      var d = new Date(date);
      d.setDate(d.getDate() + days);
      return d;
    }

    function formatDate(d) {
      return d.toISOString().split('T')[0];
    }

    function getPredictions() {
      if (!settings.lastPeriodDate) return [];
      var start = new Date(settings.lastPeriodDate + 'T00:00:00');
      var cycles = [];
      var ovulationOffset = settings.cycleLength - 14;

      for (var i = 0; i < 6; i++) {
        var cycleStart = addDays(start, i * settings.cycleLength);
        var cycleEnd = addDays(cycleStart, settings.periodLength - 1);
        var ovulation = addDays(cycleStart, ovulationOffset);
        var fertileStart = addDays(ovulation, -5);
        var fertileEnd = addDays(ovulation, 1);
        cycles.push({ 
          cycleStart: cycleStart, 
          cycleEnd: cycleEnd, 
          ovulation: ovulation, 
          fertileStart: fertileStart, 
          fertileEnd: fertileEnd 
        });
      }
      return cycles;
    }

    function getDayType(dateStr) {
      var date = new Date(dateStr + 'T00:00:00');
      var predictions = getPredictions();
      var isPeriod = false;
      var isOvulation = false;
      var isFertile = false;

      // Check manual period days first
      if (manualPeriodDays[dateStr]) {
        isPeriod = true;
      } else {
        // Check predicted period days
        for (var i = 0; i < predictions.length; i++) {
          var pred = predictions[i];
          if (date >= pred.cycleStart && date <= pred.cycleEnd) isPeriod = true;
          if (formatDate(date) === formatDate(pred.ovulation)) isOvulation = true;
          if (date >= pred.fertileStart && date <= pred.fertileEnd) isFertile = true;
        }
      }

      return { isPeriod: isPeriod, isOvulation: isOvulation, isFertile: isFertile };
    }

    function updateCycleInfo() {
      if (!settings.lastPeriodDate) {
        document.getElementById('cycleInfo').innerHTML = '<p>Please enter your last period start date to see cycle information.</p>';
        return;
      }

      var lastPeriod = new Date(settings.lastPeriodDate + 'T00:00:00');
      var today = new Date();
      today.setHours(0, 0, 0, 0);
      
      var daysSinceLastPeriod = Math.floor((today - lastPeriod) / (1000 * 60 * 60 * 24));
      var currentCycleDay = (daysSinceLastPeriod % settings.cycleLength) + 1;
      
      var html = '<h4>📊 Current Cycle Information</h4>';
      html += '<p><strong>Last Period Started:</strong> ' + formatDate(lastPeriod) + '</p>';
      html += '<p><strong>Current Cycle Day:</strong> ' + currentCycleDay + ' of ' + settings.cycleLength + '</p>';
      html += '<p><strong>Days Since Last Period:</strong> ' + daysSinceLastPeriod + ' days</p>';
      
      document.getElementById('cycleInfo').innerHTML = html;
    }

    function updatePredictions() {
      var predictions = getPredictions();
      var today = new Date();
      today.setHours(0, 0, 0, 0);

      var nextPeriod = null;
      var nextOvulation = null;

      for (var i = 0; i < predictions.length; i++) {
        if (!nextPeriod && predictions[i].cycleStart > today) {
          nextPeriod = predictions[i];
        }
        if (!nextOvulation && predictions[i].ovulation > today) {
          nextOvulation = predictions[i];
        }
      }

      var html = '<h3>📅 Upcoming Events</h3>';
      if (nextPeriod) {
        var daysUntil = Math.floor((nextPeriod.cycleStart - today) / (1000 * 60 * 60 * 24));
        html += '<p><strong>Next Period:</strong> ' + formatDate(nextPeriod.cycleStart) + ' (' + daysUntil + ' days)</p>';
      }
      if (nextOvulation) {
        var daysUntilOv = Math.floor((nextOvulation.ovulation - today) / (1000 * 60 * 60 * 24));
        html += '<p><strong>Next Ovulation:</strong> ' + formatDate(nextOvulation.ovulation) + ' (' + daysUntilOv + ' days)</p>';
      }
      if (nextPeriod) {
        html += '<p><strong>Fertile Window:</strong> ' + formatDate(nextPeriod.fertileStart) + ' to ' + formatDate(nextPeriod.fertileEnd) + '</p>';
      }
      html += '<p style="font-size: 0.85rem; color: #666; margin-top: 10px;">⚠️ These are estimates based on your cycle data.</p>';

      document.getElementById('predictions').innerHTML = html;
    }

    function renderCalendar() {
      var today = new Date();
      var displayMonth = new Date(today.getFullYear(), today.getMonth() + monthOffset, 1);
      var year = displayMonth.getFullYear();
      var month = displayMonth.getMonth();

      document.getElementById('monthName').textContent = 
        displayMonth.toLocaleString('default', { month: 'long', year: 'numeric' });

      var firstDay = new Date(year, month, 1);
      var lastDay = new Date(year, month + 1, 0);
      var daysInMonth = lastDay.getDate();
      var startingDay = firstDay.getDay();

      var html = '';
      for (var i = 0; i < startingDay; i++) {
        html += '<div></div>';
      }

      for (var day = 1; day <= daysInMonth; day++) {
        var dateStr = formatDate(new Date(year, month, day));
        var dayInfo = getDayType(dateStr);
        var note = notes[dateStr] || '';
        var isToday = dateStr === formatDate(new Date());

        var classes = 'day';
        if (isToday) classes += ' today';
        if (dayInfo.isPeriod) classes += ' period';
        else if (dayInfo.isOvulation) classes += ' ovulation';
        else if (dayInfo.isFertile) classes += ' fertile';

        html += '<div class="' + classes + '" onclick="selectDate(\'' + dateStr + '\')" oncontextmenu="quickTogglePeriod(\'' + dateStr + '\'); return false;">';
        html += '<div class="day-num">' + day + '</div>';
        if (dayInfo.isPeriod) {
          var periodTag = manualPeriodDays[dateStr] ? '🩸 Marked' : 'Period';
          html += '<span class="day-tag">' + periodTag + '</span>';
        }
        if (dayInfo.isOvulation) html += '<span class="day-tag" style="background: #03a9f4;">Ovulation</span>';
        if (note) html += '<div class="day-note">' + note + '</div>';
        html += '</div>';
      }

      document.getElementById('calendarDays').innerHTML = html;
    }

    function changeMonth(delta) {
      monthOffset += delta;
      renderCalendar();
    }

    function selectDate(dateStr) {
      selectedDate = dateStr;
      document.getElementById('noteEditor').classList.remove('hidden');
      document.getElementById('noteDate').textContent = 'Notes for ' + dateStr;
      document.getElementById('noteText').value = notes[dateStr] || '';
      
      // Update period toggle button text
      var toggleBtn = document.getElementById('periodToggleText');
      if (manualPeriodDays[dateStr]) {
        toggleBtn.textContent = '❌ Unmark Period';
      } else {
        toggleBtn.textContent = '🩸 Mark as Period';
      }
    }

    function saveNote() {
      var text = document.getElementById('noteText').value;
      if (text.trim()) {
        notes[selectedDate] = text;
      } else {
        delete notes[selectedDate];
      }
      saveData();
      renderCalendar();
      alert('Note saved!');
    }

    function addSymptomEntry() {
      var date = document.getElementById('symptomDate').value;
      if (!date) return;
      
      if (!symptoms[date]) {
        symptoms[date] = { mood: 3, energy: 3, pain: 1 };
        saveData();
      }
      renderSymptoms();
    }

    function updateSymptom(date, field, value) {
      symptoms[date][field] = parseInt(value);
      saveData();
    }

    function renderSymptoms() {
      var dates = Object.keys(symptoms).sort().reverse().slice(0, 10);
      
      var html = '';
      if (dates.length === 0) {
        html = '<p style="color: #666;">No symptoms tracked yet. Select a date above to start.</p>';
      } else {
        for (var i = 0; i < dates.length; i++) {
          var date = dates[i];
          var data = symptoms[date];
          html += '<div class="symptom-entry">';
          html += '<div class="symptom-date">' + date + '</div>';
          html += '<label><strong>😊 Mood (1-5):</strong> ' + data.mood;
          html += '<input type="range" min="1" max="5" value="' + data.mood + '" ';
          html += 'onchange="updateSymptom(\'' + date + '\', \'mood\', this.value); renderSymptoms();"></label>';
          html += '<label><strong>⚡ Energy (1-5):</strong> ' + data.energy;
          html += '<input type="range" min="1" max="5" value="' + data.energy + '" ';
          html += 'onchange="updateSymptom(\'' + date + '\', \'energy\', this.value); renderSymptoms();"></label>';
          html += '<label><strong>💧 Pain Level (1-5):</strong> ' + data.pain;
          html += '<input type="range" min="1" max="5" value="' + data.pain + '" ';
          html += 'onchange="updateSymptom(\'' + date + '\', \'pain\', this.value); renderSymptoms();"></label>';
          html += '</div>';
        }
      }
      
      document.getElementById('symptomEntries').innerHTML = html;
    }

    function exportData() {
      var dataStr = JSON.stringify({ settings: settings, notes: notes, symptoms: symptoms, manualPeriodDays: manualPeriodDays }, null, 2);
      var dataBlob = new Blob([dataStr], { type: 'application/json' });
      var url = URL.createObjectURL(dataBlob);
      var link = document.createElement('a');
      link.href = url;
      link.download = 'cycle-tracker-backup-' + new Date().toISOString().split('T')[0] + '.json';
      link.click();
      alert('Data exported successfully!');
    }

    function importData(event) {
      var file = event.target.files[0];
      if (!file) return;

      var reader = new FileReader();
      reader.onload = function(e) {
        try {
          var data = JSON.parse(e.target.result);
          if (data.settings) settings = data.settings;
          if (data.notes) notes = data.notes;
          if (data.symptoms) symptoms = data.symptoms;
          if (data.manualPeriodDays) manualPeriodDays = data.manualPeriodDays;
          
          saveData();
          loadData();
          renderCalendar();
          renderSymptoms();
          updatePredictions();
          updateCycleInfo();
          
          alert('Data imported successfully!');
        } catch (err) {
          alert('Error importing data. Please check the file format.');
        }
      };
      reader.readAsText(file);
    }

    function toggleMarkMode() {
      markPeriodMode = !markPeriodMode;
      var btn = document.getElementById('markPeriodBtn');
      if (markPeriodMode) {
        btn.textContent = '✅ Done Marking';
        btn.style.background = '#4caf50';
        alert('Period marking mode ON. Click on days to mark/unmark period days.');
      } else {
        btn.textContent = '🩸 Mark Period Days';
        btn.style.background = '#ff4081';
      }
    }

    function togglePeriodDay(dateStr) {
      if (manualPeriodDays[dateStr]) {
        delete manualPeriodDays[dateStr];
      } else {
        manualPeriodDays[dateStr] = true;
      }
      saveData();
      renderCalendar();
      
      // Update button text if note editor is open
      if (selectedDate === dateStr && !document.getElementById('noteEditor').classList.contains('hidden')) {
        var toggleBtn = document.getElementById('periodToggleText');
        if (manualPeriodDays[dateStr]) {
          toggleBtn.textContent = '❌ Unmark Period';
        } else {
          toggleBtn.textContent = '🩸 Mark as Period';
        }
      }
    }

    function quickTogglePeriod(dateStr) {
      togglePeriodDay(dateStr);
    }

    function showTab(tabName) {
      var tabs = document.querySelectorAll('.tab');
      for (var i = 0; i < tabs.length; i++) {
        tabs[i].classList.remove('active');
      }
      
      var tabContents = document.querySelectorAll('[id$="-tab"]');
      for (var i = 0; i < tabContents.length; i++) {
        tabContents[i].classList.add('hidden');
      }
      
      event.target.classList.add('active');
      document.getElementById(tabName + '-tab').classList.remove('hidden');

      if (tabName === 'calendar') renderCalendar();
      if (tabName === 'symptoms') renderSymptoms();
      if (tabName === 'settings') {
        updatePredictions();
        updateCycleInfo();
      }
    }

    window.onload = function() {
      loadData();
      renderCalendar();
      updatePredictions();
      updateCycleInfo();
    };
  </script>
</body>
</html>
