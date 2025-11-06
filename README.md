<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>Blossom — Standalone</title>
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <!-- CDN: React, ReactDOM, Babel for JSX transpile, Chart.js -->
  <script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <!-- Babel for in-browser JSX transpilation (dev convenience) -->
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

  <style>
    /* ---------- Basic app styles (kept simple & responsive) ---------- */
    :root{
      --bg: #fff;
      --card: #fff;
      --text: #222;
      --muted: #666;
      --accent-from: #ff6b9d;
      --accent-to: #f72585;
      --accent: linear-gradient(135deg,var(--accent-from) 0%,var(--accent-to) 100%);
    }
    [data-theme='dark']{
      --bg: #0b1020;
      --card: #0f1724;
      --text: #e6eef8;
      --muted: #9aa7be;
    }

    html,body,#root{height:100%;}
    body{
      margin:0;
      font-family: system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
      background: linear-gradient(to bottom, #fdf2f8, #fce7f3, #ffffff);
      color:var(--text);
    }

    .container{
      max-width:1100px;
      margin:24px auto;
      padding:20px;
    }

    header.app-header{
      text-align:center;
      padding:28px 20px;
      background:var(--accent);
      border-radius:20px;
      color:white;
      margin-bottom:24px;
      box-shadow: 0 10px 40px rgba(255,107,157,0.18);
    }

    nav.tabs{
      display:flex;
      gap:10px;
      margin-bottom:24px;
      background:var(--card);
      padding:8px;
      border-radius:12px;
      box-shadow: 0 2px 12px rgba(0,0,0,0.06);
    }
    nav.tabs button{flex:1;padding:10px;border-radius:10px;border:none;cursor:pointer;background:transparent;}
    nav.tabs button.active{background:var(--accent);color:white;font-weight:600;}

    .card{
      background:var(--card);
      padding:20px;
      border-radius:16px;
      box-shadow: 0 6px 20px rgba(0,0,0,0.06);
      margin-bottom:20px;
    }

    .grid{
      display:grid;
      gap:16px;
    }

    @media(min-width:900px){
      .grid.cols-2{grid-template-columns: 1fr 1fr;}
      .grid.cols-3{grid-template-columns: repeat(3,1fr);}
    }

    input,textarea,select{
      width:100%;
      padding:10px;
      border:1px solid #e9ecef;
      border-radius:10px;
      box-sizing:border-box;
      font-family:inherit;
      font-size:1rem;
      color:var(--text);
      background:transparent;
    }
    label{display:block;margin-bottom:8px;font-weight:600;color:var(--text);}

    .calendar-grid{display:grid;grid-template-columns:repeat(7,1fr);gap:8px;margin-top:12px;}
    .day{background:var(--card);padding:10px;border-radius:10px;min-height:100px;border:1px solid #e9ecef;position:relative;display:flex;flex-direction:column;}
    .day.empty{background:transparent;border:none;min-height:0;}
    .day.today{box-shadow:0 6px 24px rgba(167,139,250,0.12);border:3px solid rgba(255,107,157,0.6);}
    .tag{font-size:0.75rem;padding:4px 8px;border-radius:999px;display:inline-block;font-weight:700;margin-bottom:6px;}
    .period-bg{background:#ffe0f0;}
    .ovulation-bg{background:#fff4e6;}
    .small{font-size:0.85rem;color:var(--muted);}

    .symptom-btn{display:flex;flex-direction:column;align-items:center;padding:12px;border-radius:10px;border:2px solid #e9ecef;background:transparent;cursor:pointer;min-width:86px;}
    .symptom-btn.selected{box-shadow:0 6px 20px rgba(0,0,0,0.06);}

    footer{margin-top:30px;text-align:center;color:var(--muted);}
    .flex{display:flex;gap:12px;align-items:center;}
    .grow{flex:1;}
    .muted{color:var(--muted);}
    .pill{padding:6px 10px;border-radius:999px;background:rgba(255,255,255,0.2);display:inline-block;}
    .btn{padding:10px 14px;border-radius:10px;border:none;background:var(--accent);color:white;cursor:pointer;}
    .ghost{background:transparent;border:1px solid #e9ecef;color:var(--text);}
  </style>
</head>
<body>
  <div id="root"></div>

  <!-- Main app script (JSX via Babel) -->
  <script type="text/babel">

  const { useState, useEffect, useMemo, useRef } = React;

  /*****************************************************************
   * Utility helpers
   *****************************************************************/
  function addDays(date, days){
    const d = new Date(date);
    d.setDate(d.getDate() + days);
    return d;
  }
  function formatISO(d){ // YYYY-MM-DD
    const dt = new Date(d);
    return dt.toISOString().split('T')[0];
  }
  function formatPretty(d){
    return new Date(d).toLocaleDateString('en-US', { month:'short', day:'numeric' });
  }

  /*****************************************************************
   * getCyclePhase(profile, date)
   * - returns one of: 'menstrual', 'follicular', 'ovulation', 'luteal' or null
   * - based on profile.lastPeriodStart, profile.cycleLength, profile.periodLength
   *****************************************************************/
  function getCyclePhase(profile, date){
    if(!profile.lastPeriodStart) return null;
    const start = new Date(profile.lastPeriodStart);
    const current = new Date(date);
    const daysSinceStart = Math.floor((current - start) / (1000*60*60*24));
    const dayInCycle = ((daysSinceStart % profile.cycleLength) + profile.cycleLength) % profile.cycleLength;

    if(dayInCycle < profile.periodLength) return 'menstrual';
    if(dayInCycle < 14) return 'follicular';
    if(dayInCycle >= 14 && dayInCycle < 16) return 'ovulation';
    return 'luteal';
  }

  /*****************************************************************
   * PhaseCard component
   * - Visual card for each menstrual phase
   * - Props: phase (string), isActive (bool)
   * - This component is presentational only.
   *****************************************************************/
  function PhaseCard({ phase, isActive }){
    const phases = {
      menstrual: { title:'Menstrual Phase', emoji:'🩸', color:'#ff6b9d', desc:'Your period. Time for self-care and rest.', tips:['Stay hydrated','Gentle exercise','Iron-rich foods'] },
      follicular: { title:'Follicular Phase', emoji:'✨', color:'#a78bfa', desc:'Energy rising. Great time to start new projects.', tips:['High energy workouts','Social activities','Try new things'] },
      ovulation: { title:'Ovulation Phase', emoji:'☀️', color:'#fbbf24', desc:'Peak fertility. You might feel most confident.', tips:['Most fertile days','High energy','Social peak'] },
      luteal: { title:'Luteal Phase', emoji:'🌙', color:'#60a5fa', desc:'Winding down. Focus on self-care.', tips:['Relaxing activities','Comfort foods','Early nights'] }
    };
    const info = phases[phase];
    if(!info) return null;

    const bg = isActive ? `linear-gradient(135deg, ${info.color} 0%, #7c3aed 100%)` : 'transparent';
    const color = isActive ? '#fff' : '#222';

    return (
      <div className="card" style={{flex:1,minWidth:200,background:bg,color:color,transform:isActive?'scale(1.02)':'none'}}>
        <div style={{fontSize:40, marginBottom:8}}>{info.emoji}</div>
        <h3 style={{margin:'0 0 8px 0'}}>{info.title}</h3>
        <p style={{margin:'0 0 12px',opacity:0.95}}>{info.desc}</p>
        <div style={{fontSize:13,opacity:0.9}}>
          {info.tips.map((t,i)=> <div key={i}>• {t}</div>)}
        </div>
      </div>
    );
  }

  /*****************************************************************
   * SymptomButton component
   * - Small button representing a symptom
   * - Props: icon (node), label (string), isSelected (bool), onClick, color
   *****************************************************************/
  function SymptomButton({ icon, label, isSelected, onClick, color }){
    return (
      <button className={`symptom-btn ${isSelected ? 'selected' : ''}`} onClick={onClick}
        style={{
          borderColor: isSelected ? color : '#e9ecef',
          color: isSelected ? color : 'inherit',
          background: isSelected ? `${color}22` : 'transparent'
        }}>
        <div style={{fontSize:20}}>{icon}</div>
        <div style={{fontSize:13, marginTop:6, fontWeight:isSelected?700:500}}>{label}</div>
      </button>
    );
  }

  /*****************************************************************
   * Calendar component
   * - Shows a month calendar
   * - Allows notes per day (saves via setNotes prop)
   * - Highlights period days and fertile window based on predictions
   * - Props: profile, notes, setNotes, periodHistory
   *
   * Explanations inline: we compute `predictions` from periodHistory (if available)
   * to find cycle starts and ovulation dates.
   *****************************************************************/
  function Calendar({ profile, notes, setNotes, periodHistory }){
    const [monthOffset, setMonthOffset] = useState(0);
    const today = new Date();
    const displayMonth = new Date(today.getFullYear(), today.getMonth() + monthOffset, 1);

    // ovulation offset typically cycleLength - 14
    const ovulationOffset = profile.cycleLength - 14;

    // Build predictions: use periodHistory if available (array of ISO dates)
    const predictions = useMemo(()=>{
      if(!profile.lastPeriodStart && (!periodHistory || periodHistory.length === 0)) return [];
      // use the last known period start or periodHistory
      const starts = (periodHistory && periodHistory.length > 0) ? periodHistory.slice().sort() : [profile.lastPeriodStart];
      const cycles = [];
      // for each recent start, predict adjacent cycles
      starts.forEach(sStart => {
        const startDate = new Date(sStart);
        for(let i=-1;i<4;i++){
          const cs = addDays(startDate, i*profile.cycleLength);
          const ov = addDays(cs, ovulationOffset);
          cycles.push({ cycleStart: cs, ovulation: ov });
        }
      });
      return cycles;
    }, [profile, periodHistory]);

    // build days for visible month
    const days = [];
    const year = displayMonth.getFullYear();
    const month = displayMonth.getMonth();
    const firstDay = new Date(year, month, 1).getDay(); // 0..6
    for(let i=0;i<firstDay;i++) days.push(null);
    for(let i=1;i<=31;i++){
      const d = new Date(year, month, i);
      if(d.getMonth() !== month) break;
      days.push(formatISO(d));
    }

    const weekDays = ['Sun','Mon','Tue','Wed','Thu','Fri','Sat'];

    const onNoteChange = (iso, text) => {
      setNotes(prev => {
        const next = {...prev};
        next[iso] = { text };
        return next;
      });
    };

    return (
      <section className="card">
        <div style={{display:'flex',alignItems:'center',justifyContent:'space-between',marginBottom:12}}>
          <div style={{display:'flex',alignItems:'center',gap:12}}>
            <div style={{fontSize:20}}>📅</div>
            <h2 style={{margin:0}}>Your Cycle Calendar</h2>
          </div>
          <div>
            <button className="ghost" onClick={()=>setMonthOffset(m => m-1)} style={{marginRight:8}}>← Prev</button>
            <span className="small" style={{fontWeight:700}}>{displayMonth.toLocaleString('default',{month:'long'})} {year}</span>
            <button className="ghost" onClick={()=>setMonthOffset(m => m+1)} style={{marginLeft:8}}>Next →</button>
          </div>
        </div>

        <div style={{display:'grid',gridTemplateColumns:'repeat(7,1fr)',gap:6,marginBottom:6}}>
          {weekDays.map(d => <div key={d} style={{textAlign:'center',fontWeight:700}}>{d}</div>)}
        </div>

        <div className="calendar-grid">
          {days.map((iso, idx) => {
            if(!iso) return <div key={'e'+idx} className="day empty"></div>;
            const date = new Date(iso);
            const note = notes[iso]?.text || '';
            const isToday = formatISO(today) === iso;
            const isOvulation = predictions.some(p => formatISO(p.ovulation) === iso);
            const isPeriod = predictions.some(p => {
              const startISO = formatISO(p.cycleStart);
              const start = p.cycleStart;
              const end = addDays(start, profile.periodLength);
              return date >= start && date < end;
            });
            const bgStyle = isPeriod ? {background:'#ffe0f0'} : isOvulation ? {background:'#fff4e6'} : {};
            return (
              <div key={iso} className={`day ${isToday ? 'today' : ''}`} style={{...bgStyle, border: isToday ? '3px solid rgba(255,107,157,0.6)' : '1px solid #e9ecef'}}>
                <div style={{display:'flex',justifyContent:'space-between',alignItems:'center',marginBottom:8}}>
                  <div style={{fontWeight:700}}>{date.getDate()}</div>
                  <div style={{fontSize:12}}>{isPeriod ? <span className="tag" style={{background:'#ff6b9d',color:'#fff'}}>Period</span> : isOvulation ? <span className="tag" style={{background:'#fbbf24',color:'#78350f'}}>Fertile</span> : null}</div>
                </div>

                <textarea placeholder="Add note..." value={note} onChange={(e)=>onNoteChange(iso,e.target.value)} />
              </div>
            );
          })}
        </div>

        <div style={{marginTop:16,display:'flex',gap:12,flexWrap:'wrap'}}>
          <div className="flex" style={{alignItems:'center',gap:8}}>
            <div style={{width:18,height:18,background:'#ffe0f0',border:'1px solid #ff6b9d',borderRadius:4}}></div>
            <div className="small">Period Days</div>
          </div>
          <div className="flex" style={{alignItems:'center',gap:8}}>
            <div style={{width:18,height:18,background:'#fff4e6',border:'1px solid #fbbf24',borderRadius:4}}></div>
            <div className="small">Fertile Window</div>
          </div>
          <div className="flex" style={{alignItems:'center',gap:8}}>
            <div style={{width:18,height:18,background:'white',border:'3px solid rgba(255,107,157,0.6)',borderRadius:4}}></div>
            <div className="small">Today</div>
          </div>
        </div>
      </section>
    );
  }

  /*****************************************************************
   * Symptoms component
   * - Track mood and symptoms for a selected date
   * - Renders a Chart.js line chart for mood history
   * - Props: symptoms, setSymptoms, profile
   *
   * Explanation: We store symptoms as { 'YYYY-MM-DD': { mood:1..5, cramps:true, ... } }
   *****************************************************************/
  function Symptoms({ symptoms, setSymptoms, profile }){
    const [selectedDate, setSelectedDate] = useState('');
    const canvasRef = useRef(null);
    const chartRef = useRef(null);

    const currentSymptoms = selectedDate ? (symptoms[selectedDate] || {}) : {};

    const symptomOptions = [
      { key:'cramps', label:'Cramps', icon:'⚡', color:'#ef4444' },
      { key:'bloating', label:'Bloating', icon:'☁️', color:'#f59e0b' },
      { key:'headache', label:'Headache', icon:'🤕', color:'#8b5cf6' },
      { key:'tender', label:'Tender Breasts', icon:'💗', color:'#ec4899' },
      { key:'fatigue', label:'Fatigue', icon:'🌙', color:'#6366f1' },
      { key:'acne', label:'Acne', icon:'✨', color:'#14b8a6' }
    ];

    // Build / update chart whenever symptoms change
    useEffect(()=>{
      if(chartRef.current){
        chartRef.current.destroy();
        chartRef.current = null;
      }
      const ctx = canvasRef.current.getContext('2d');
      const dates = Object.keys(symptoms).sort();
      const moods = dates.map(d => symptoms[d]?.mood || 3);

      chartRef.current = new Chart(ctx, {
        type:'line',
        data:{
          labels: dates.length>0 ? dates.map(d=> new Date(d).toLocaleDateString('en-US',{month:'short',day:'numeric'})) : ['No data'],
          datasets:[
            {
              label:'Mood',
              data: moods.length>0 ? moods : [3],
              borderColor:'#ff6b9d',
              backgroundColor:'rgba(255,107,157,0.12)',
              tension:0.4,
              fill:true,
              pointRadius:6,
              pointBackgroundColor:'#ff6b9d'
            }
          ]
        },
        options:{
          responsive:true,
          maintainAspectRatio:false,
          scales:{
            y:{
              beginAtZero:true,
              suggestedMax:5,
              ticks:{
                stepSize:1,
                callback: function(value){ const moods = ['','😢','😕','😐','🙂','😊']; return moods[value] || value; }
              }
            },
            x:{ grid:{ display:false } }
          },
          plugins:{
            legend:{ display:false },
            tooltip:{ backgroundColor:'#fff', titleColor:'#333', bodyColor:'#666', borderColor:'#ff6b9d', borderWidth:1 }
          }
        }
      });

      return ()=>{ if(chartRef.current) chartRef.current.destroy(); };

    }, [symptoms]);

    const toggleSymptom = (key) => {
      if(!selectedDate) return alert('Select a date first');
      setSymptoms(prev=>{
        const next = {...prev};
        next[selectedDate] = {...(next[selectedDate]||{})};
        next[selectedDate][key] = !next[selectedDate][key];
        return next;
      });
    };

    const setMood = (value) => {
      if(!selectedDate) return alert('Select a date first');
      setSymptoms(prev=>{
        const next = {...prev};
        next[selectedDate] = {...(next[selectedDate]||{}), mood:value};
        return next;
      });
    };

    const deleteEntry = (date) => {
      setSymptoms(prev=>{
        const next = {...prev};
        delete next[date];
        return next;
      });
    };

    return (
      <section className="card">
        <div style={{display:'flex',alignItems:'center',gap:10,marginBottom:12}}>
          <div style={{fontSize:20}}>📈</div>
          <h2 style={{margin:0}}>Track Your Symptoms</h2>
        </div>

        <div style={{marginBottom:18}}>
          <label>Select Date</label>
          <input type="date" value={selectedDate} onChange={(e)=>setSelectedDate(e.target.value)} max={formatISO(new Date())} />
        </div>

        {selectedDate && (
          <>
            <div style={{marginBottom:18}}>
              <h3 style={{margin:'0 0 8px 0'}}>How are you feeling?</h3>
              <div style={{display:'flex',gap:8}}>
                {[1,2,3,4,5].map(val => (
                  <button key={val} onClick={()=>setMood(val)}
                    style={{
                      flex:1,fontSize:22,padding:12,borderRadius:10,border:(currentSymptoms.mood||3)===val ? '3px solid #ff6b9d' : '1px solid #e9ecef',background:(currentSymptoms.mood||3)===val ? '#fff0f5' : 'transparent'
                    }}>
                    {['😢','😕','😐','🙂','😊'][val-1]}
                  </button>
                ))}
              </div>
            </div>

            <div style={{marginBottom:18}}>
              <h3 style={{margin:'0 0 8px 0'}}>Symptoms</h3>
              <div style={{display:'grid',gridTemplateColumns:'repeat(auto-fit,minmax(100px,1fr))',gap:10}}>
                {symptomOptions.map(opt => (
                  <SymptomButton key={opt.key} icon={opt.icon} label={opt.label}
                    isSelected={!!currentSymptoms[opt.key]} onClick={()=>toggleSymptom(opt.key)} color={opt.color} />
                ))}
              </div>
            </div>
          </>
        )}

        <div style={{marginTop:12}}>
          <h3 style={{margin:'0 0 8px 0'}}>Mood Trends</h3>
          <div style={{height:260,padding:12,borderRadius:10,background:'transparent'}}>
            <canvas ref={canvasRef}></canvas>
          </div>
        </div>

        {Object.keys(symptoms).length>0 && (
          <div style={{marginTop:12}}>
            <h3 style={{margin:'0 0 8px 0'}}>Recent Entries</h3>
            <div style={{maxHeight:260,overflowY:'auto'}}>
              {Object.entries(symptoms).sort((a,b)=>b[0].localeCompare(a[0])).slice(0,20).map(([date,data])=>(
                <div key={date} style={{padding:10,background:'#f8f9fa',borderRadius:8,display:'flex',justifyContent:'space-between',alignItems:'center',marginBottom:8}}>
                  <div>
                    <strong>{new Date(date).toLocaleDateString('en-US',{month:'short',day:'numeric',year:'numeric'})}</strong>
                    <div className="small">Mood: {['😢','😕','😐','🙂','😊'][data.mood -1] || '😐'} { /* list symptoms */ }
                      {['cramps','bloating','headache','tender','fatigue','acne'].filter(k => data[k]).length>0 && (
                        <span style={{marginLeft:8}}>• {['cramps','bloating','headache','tender','fatigue','acne'].filter(k=>data[k]).join(', ')}</span>
                      )}
                    </div>
                  </div>
                  <div>
                    <button className="ghost" onClick={()=>deleteEntry(date)} style={{color:'#dc2626'}}>Delete</button>
                  </div>
                </div>
              ))}
            </div>
          </div>
        )}
      </section>
    );
  }

  /*****************************************************************
   * App root component
   * - Handles top-level state, persistence (localStorage), dark mode,
   *   notifications, period history management and predictions.
   *
   * Explanation of storage:
   *   We persist everything under localStorage key 'blossom-data'
   *   Structure:
   *     {
   *       profile: {cycleLength, periodLength, lastPeriodStart},
   *       notes: { 'YYYY-MM-DD': {text} },
   *       symptoms: { 'YYYY-MM-DD': {...} },
   *       periodHistory: ['YYYY-MM-DD', ...],
   *       settings: { theme:'light'|'dark', remindersEnabled:true|false, reminderHour: '08:00' }
   *     }
   *****************************************************************/
  function App(){
    // default profile
    const [profile, setProfile] = useState({ cycleLength:28, periodLength:5, lastPeriodStart:'' });
    const [notes, setNotes] = useState({});
    const [symptoms, setSymptoms] = useState({});
    const [periodHistory, setPeriodHistory] = useState([]); // user can add multiple start dates to improve predictions
    const [loading, setLoading] = useState(true);
    const [tab, setTab] = useState('home');
    const [settings, setSettings] = useState({ theme:'light', remindersEnabled:false, reminderHour:'09:00' });

    const STORAGE_KEY = 'blossom-data-v1';

    // Load persisted data on mount
    useEffect(()=>{
      try{
        const raw = localStorage.getItem(STORAGE_KEY);
        if(raw){
          const parsed = JSON.parse(raw);
          setProfile(parsed.profile || {cycleLength:28,periodLength:5,lastPeriodStart:''});
          setNotes(parsed.notes || {});
          setSymptoms(parsed.symptoms || {});
          setPeriodHistory(parsed.periodHistory || []);
          setSettings(parsed.settings || {theme:'light',remindersEnabled:false,reminderHour:'09:00'});
          // apply theme
          if(parsed.settings && parsed.settings.theme === 'dark') document.documentElement.setAttribute('data-theme','dark');
        }
      }catch(err){ console.error('Failed to load data', err); }
      setLoading(false);
    },[]);

    // Save whenever main data changes
    useEffect(()=>{
      if(loading) return;
      const payload = { profile, notes, symptoms, periodHistory, settings };
      localStorage.setItem(STORAGE_KEY, JSON.stringify(payload));
    }, [profile, notes, symptoms, periodHistory, settings, loading]);

    // compute current phase based on today's date
    const currentPhase = useMemo(()=> getCyclePhase(profile, formatISO(new Date())), [profile]);

    // compute nextPeriod from periodHistory (prefer history if available)
    const nextPeriod = useMemo(()=>{
      const lastStart = periodHistory && periodHistory.length>0 ? new Date(periodHistory.slice().sort().reverse()[0]) : (profile.lastPeriodStart ? new Date(profile.lastPeriodStart) : null);
      if(!lastStart) return null;
      const today = new Date();
      const daysSinceStart = Math.floor((today - lastStart) / (1000*60*60*24));
      const cyclesPassed = Math.floor(daysSinceStart / profile.cycleLength);
      const next = addDays(lastStart, (cyclesPassed + 1) * profile.cycleLength);
      const daysUntil = Math.ceil((next - today) / (1000*60*60*24));
      return { date: next, daysUntil };
    }, [profile, periodHistory]);

    // Calculate average cycle length from periodHistory (if 2+ starts)
    const averageCycle = useMemo(()=>{
      if(!periodHistory || periodHistory.length < 2) return profile.cycleLength;
      const sorted = periodHistory.slice().sort();
      const diffs = [];
      for(let i=1;i<sorted.length;i++){
        const a = new Date(sorted[i-1]);
        const b = new Date(sorted[i]);
        const diff = Math.round((b - a) / (1000*60*60*24));
        diffs.push(diff);
      }
      const avg = Math.round(diffs.reduce((s,x)=>s+x,0)/diffs.length);
      return avg;
    }, [periodHistory, profile.cycleLength]);

    // Allow toggling theme and persist
    const toggleTheme = () => {
      const next = settings.theme === 'dark' ? 'light' : 'dark';
      setSettings(s => ({...s, theme: next}));
      if(next === 'dark') document.documentElement.setAttribute('data-theme','dark');
      else document.documentElement.removeAttribute('data-theme');
    };

    // Notifications: basic in-page reminder (only works when app open)
    useEffect(()=>{
      let timer = null;
      if(settings.remindersEnabled){
        // check every minute for the reminder time
        timer = setInterval(()=>{
          const now = new Date();
          const hhmm = now.toTimeString().slice(0,5);
          if(hhmm === settings.reminderHour){
            // show notification
            if(window.Notification && Notification.permission === 'granted'){
              new Notification('Blossom reminder', { body: 'Time to log how you feel today 🌸' });
            }
            // also alert fallback
            else {
              console.log('Reminder: time to log today');
            }
          }
        }, 60*1000);
      }
      return ()=> { if(timer) clearInterval(timer); };
    }, [settings.remindersEnabled, settings.reminderHour]);

    // Helper: request notification permission
    const requestNotificationPermission = async () => {
      if(!('Notification' in window)) return alert('Notifications not supported in this browser');
      if(Notification.permission === 'granted') return true;
      const p = await Notification.requestPermission();
      return p === 'granted';
    };

    // Add a period start entry to history and update profile.lastPeriodStart
    const addPeriodStart = (iso) => {
      if(!iso) return;
      setPeriodHistory(prev => {
        const next = Array.from(new Set([...(prev||[]), iso])); // unique
        next.sort();
        return next;
      });
      setProfile(prev => ({...prev, lastPeriodStart: iso }));
      alert('Period start added to history.');
    };

    // Remove a history entry
    const removeHistory = (iso) => {
      setPeriodHistory(prev => (prev||[]).filter(d=>d!==iso));
    };

    // quick export / import (JSON)
    const exportData = () => {
      const payload = { profile, notes, symptoms, periodHistory, settings };
      const blob = new Blob([JSON.stringify(payload, null, 2)], { type:'application/json' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url; a.download = 'blossom-data.json'; a.click();
      URL.revokeObjectURL(url);
    };
    const importData = (file) => {
      const reader = new FileReader();
      reader.onload = () => {
        try{
          const parsed = JSON.parse(reader.result);
          setProfile(parsed.profile || profile);
          setNotes(parsed.notes || {});
          setSymptoms(parsed.symptoms || {});
          setPeriodHistory(parsed.periodHistory || []);
          setSettings(parsed.settings || settings);
          alert('Data imported');
        }catch(err){ alert('Invalid file'); }
      };
      reader.readAsText(file);
    };

    if(loading) return <div className="container" style={{textAlign:'center', paddingTop:60}}><div style={{fontSize:48}}>🌸</div><p className="small">Loading Blossom…</p></div>;

    return (
      <div className="container">
        <header className="app-header">
          <div style={{fontSize:44}}>🌸</div>
          <h1 style={{margin:'8px 0 4px 0'}}>Blossom</h1>
          <div style={{opacity:0.95}}>Your Personal Health Companion</div>

          {nextPeriod && (
            <div style={{marginTop:12,display:'inline-block',padding:'10px 16px',background:'rgba(255,255,255,0.18)',borderRadius:12}}>
              <div className="small">Next period in</div>
              <div style={{fontSize:20,fontWeight:700}}>{nextPeriod.daysUntil} days</div>
              <div className="small">{nextPeriod.date.toLocaleDateString('en-US',{month:'long',day:'numeric'})}</div>
            </div>
          )}
        </header>

        <nav className="tabs">
          {[
            { id:'home', label:'Overview', icon:'💖' },
            { id:'calendar', label:'Calendar', icon:'📅' },
            { id:'symptoms', label:'Symptoms', icon:'📈' },
            { id:'settings', label:'Settings', icon:'⚙️' }
          ].map(t => (
            <button key={t.id} className={tab===t.id? 'active' : ''} onClick={()=>setTab(t.id)}>
              <span style={{marginRight:8}}>{t.icon}</span>{t.label}
            </button>
          ))}
        </nav>

        {/* Home / Overview */}
        {tab === 'home' && (
          <>
            <section className="card">
              <div style={{display:'flex',justifyContent:'space-between',alignItems:'center',gap:12}}>
                <div>
                  <h2 style={{margin:0}}>Cycle Overview</h2>
                  <div className="small">Average cycle from history: <strong>{averageCycle} days</strong></div>
                </div>
                <div style={{textAlign:'right'}}>
                  <div className="small">Theme</div>
                  <div style={{display:'flex',gap:8,alignItems:'center'}}>
                    <button className="ghost" onClick={toggleTheme}>{settings.theme === 'dark' ? 'Light' : 'Dark'}</button>
                    <div className="small">• Reminders</div>
                    <label style={{display:'flex',alignItems:'center',gap:8}}>
                      <input type="checkbox" checked={settings.remindersEnabled} onChange={(e)=>setSettings(s=>({...s,remindersEnabled:e.target.checked}))} />
                    </label>
                  </div>
                </div>
              </div>

              <div style={{marginTop:14}} className="grid cols-2">
                <div>
                  <h3 style={{marginTop:0}}>Cycle Phases</h3>
                  <div style={{display:'flex',gap:12,flexWrap:'wrap'}}>
                    <PhaseCard phase="menstrual" isActive={currentPhase==='menstrual'} />
                    <PhaseCard phase="follicular" isActive={currentPhase==='follicular'} />
                    <PhaseCard phase="ovulation" isActive={currentPhase==='ovulation'} />
                    <PhaseCard phase="luteal" isActive={currentPhase==='luteal'} />
                  </div>
                </div>

                <div>
                  <h3 style={{marginTop:0}}>Today's Tip</h3>
                  <div style={{padding:16,borderRadius:10,background:'#f8f9fa'}}>
                    <div className="small">
                      {currentPhase === 'menstrual' && "Rest is productive. Be gentle with yourself today."}
                      {currentPhase === 'follicular' && "Energy rising! Great time to start new projects."}
                      {currentPhase === 'ovulation' && "You're in a confident phase — connect with others."}
                      {currentPhase === 'luteal' && "Winding down — focus on comfort and rest."}
                      {!currentPhase && "Start tracking your cycle to get personalized wellness insights!"}
                    </div>
                  </div>

                  <div style={{marginTop:12}}>
                    <h4 style={{margin:'8px 0'}}>Period History</h4>
                    <div style={{display:'flex',gap:8,flexWrap:'wrap',alignItems:'center'}}>
                      <input type="date" onChange={(e)=>setProfile(p=>({...p, lastPeriodStart: e.target.value}))} value={profile.lastPeriodStart || ''} />
                      <button className="btn" onClick={()=>addPeriodStart(profile.lastPeriodStart)}>Add to history</button>
                    </div>

                    <div style={{marginTop:12}}>
                      {periodHistory.length === 0 && <div className="small">No history yet — add period starts to improve predictions.</div>}
                      {periodHistory.length>0 && (
                        <div style={{marginTop:8}}>
                          {periodHistory.slice().sort((a,b)=>b.localeCompare(a)).map(d=>(
                            <div key={d} style={{display:'flex',alignItems:'center',justifyContent:'space-between',background:'#f8f9fa',padding:8,borderRadius:8,marginBottom:6}}>
                              <div>{new Date(d).toLocaleDateString()}</div>
                              <div>
                                <button className="ghost" onClick={()=>removeHistory(d)} style={{color:'#dc2626'}}>Remove</button>
                              </div>
                            </div>
                          ))}
                        </div>
                      )}
                    </div>
                  </div>
                </div>
              </div>
            </section>
          </>
        )}

        {/* Calendar */}
        {tab === 'calendar' && (
          <Calendar profile={{...profile, cycleLength: averageCycle}} notes={notes} setNotes={setNotes} periodHistory={periodHistory} />
        )}

        {/* Symptoms */}
        {tab === 'symptoms' && (
          <Symptoms symptoms={symptoms} setSymptoms={setSymptoms} profile={{...profile, cycleLength: averageCycle}} />
        )}

        {/* Settings */}
        {tab === 'settings' && (
          <section className="card">
            <h2 style={{marginTop:0}}>Settings & Data</h2>

            <div style={{display:'grid',gap:12}}>
              <div style={{background:'#f8f9fa',padding:12,borderRadius:10}}>
                <label>Last period start date</label>
                <input type="date" value={profile.lastPeriodStart || ''} onChange={(e)=>setProfile(p=>({...p, lastPeriodStart: e.target.value}))} />
              </div>

              <div style={{display:'grid',gridTemplateColumns:'1fr 1fr',gap:12}}>
                <div style={{background:'#f8f9fa',padding:12,borderRadius:10}}>
                  <label>Cycle length (days)</label>
                  <input type="number" min="21" max="35" value={profile.cycleLength} onChange={(e)=>setProfile(p=>({...p, cycleLength: Number(e.target.value)}))} />
                  <div className="small">Average from history: {averageCycle}</div>
                </div>
                <div style={{background:'#f8f9fa',padding:12,borderRadius:10}}>
                  <label>Period length (days)</label>
                  <input type="number" min="3" max="10" value={profile.periodLength} onChange={(e)=>setProfile(p=>({...p, periodLength: Number(e.target.value)}))} />
                </div>
              </div>

              <div style={{display:'flex',gap:8,alignItems:'center'}}>
                <label style={{minWidth:120}}>Reminders</label>
                <input type="time" value={settings.reminderHour} onChange={(e)=>setSettings(s=>({...s,reminderHour:e.target.value}))} />
                <label style={{display:'flex',alignItems:'center',gap:8}}>
                  <input type="checkbox" checked={settings.remindersEnabled} onChange={(e)=>setSettings(s=>({...s,remindersEnabled:e.target.checked}))} />
                  <span className="small">Enable</span>
                </label>
                <button className="ghost" onClick={async ()=>{
                  const ok = await requestNotificationPermission();
                  if(ok) alert('Notifications enabled (when allowed by browser).');
                }}>Allow browser notifications</button>
              </div>

              <div style={{display:'flex',gap:8,alignItems:'center'}}>
                <button className="btn" onClick={exportData}>Export data</button>
                <label className="ghost" style={{padding:'8px 12px',borderRadius:10,cursor:'pointer'}}>
                  Import
                  <input type="file" accept="application/json" onChange={(e)=>{ if(e.target.files && e.target.files[0]) importData(e.target.files[0]); }} style={{display:'none'}} />
                </label>
                <button className="ghost" onClick={()=>{ if(confirm('Clear all data?')) { localStorage.removeItem(STORAGE_KEY); location.reload(); } }}>Clear all</button>
              </div>

              <div style={{background:'#fff3cd',padding:12,borderRadius:10,border:'1px solid #ffc107'}}>
                <strong>About Blossom</strong>
                <div className="small">Personal, local-first cycle and symptom tracker. Data is stored in your browser.</div>
                <div className="small" style={{marginTop:6}}>Version 1.0 • Made with 💗</div>
              </div>
            </div>
          </section>
        )}

        <footer>
          <div>🌸 Take care of yourself</div>
          <div className="small">Your data is stored locally in your browser (no server).</div>
        </footer>
      </div>
    );
  }

  // Render app
  ReactDOM.createRoot(document.getElementById('root')).render(<App />);

  </script>
</body>
</html>
