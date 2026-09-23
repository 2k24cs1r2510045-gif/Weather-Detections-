<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Skyline — Weather Detection</title>
<link href="https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,400;9..144,600&family=Inter:wght@400;500;600&display=swap" rel="stylesheet">
<style>
  :root{
    --bg:#101a2e; --bg2:#182541; --card:#1b2a4a; --text:#eef2f8;
    --muted:#8ba0c2; --amber:#ffab5e; --line:rgba(255,255,255,0.09);
    padding-top:env(safe-area-inset-top,0px); padding-bottom:env(safe-area-inset-bottom,0px);
    box-sizing:border-box;
  }
  *{box-sizing:border-box;}
  html{scroll-padding-top:env(safe-area-inset-top,0px);}
  body{
    margin:0; min-height:100vh; background:radial-gradient(120% 100% at 50% 0%, var(--bg2), var(--bg));
    color:var(--text); font-family:'Inter',sans-serif; display:flex; justify-content:center; padding:48px 20px;
  }
  .app{width:100%; max-width:480px;}
  h1{font-family:'Fraunces',serif; font-weight:600; font-size:1.5rem; margin:0 0 4px; letter-spacing:-0.01em;}
  .sub{color:var(--muted); font-size:0.92rem; margin:0 0 28px;}
  .search{display:flex; gap:8px; margin-bottom:22px;}
  input{
    flex:1; background:var(--card); border:1px solid var(--line); color:var(--text);
    padding:13px 16px; border-radius:12px; font-size:0.98rem; font-family:inherit; outline:none;
  }
  input:focus{border-color:var(--amber);}
  button{
    background:var(--amber); color:#1a1200; border:none; padding:0 22px; border-radius:12px;
    font-weight:600; font-size:0.95rem; cursor:pointer; font-family:inherit;
  }
  button:active{transform:scale(0.98);}
  button:disabled{opacity:0.6; cursor:default;}
  .card{
    background:var(--card); border:1px solid var(--line); border-radius:20px; padding:32px 28px;
    min-height:180px; display:flex; flex-direction:column; justify-content:center;
  }
  .place{font-size:1.05rem; color:var(--muted); margin:0 0 2px;}
  .temp{font-family:'Fraunces',serif; font-size:4.2rem; font-weight:600; line-height:1; margin:6px 0 4px;}
  .cond{font-size:1.05rem; margin:0 0 22px; color:var(--text);}
  .grid{display:grid; grid-template-columns:repeat(3,1fr); gap:14px; border-top:1px solid var(--line); padding-top:20px;}
  .stat b{display:block; font-size:1.15rem; font-weight:600;}
  .stat span{font-size:0.78rem; color:var(--muted);}
  .empty, .error{color:var(--muted); font-size:0.95rem; line-height:1.5;}
  .error{color:#ff9d8a;}
  .loading{color:var(--muted);}
  footer{margin-top:18px; font-size:0.78rem; color:var(--muted); text-align:center;}
</style>
</head>
<body>
<div class="app">
  <h1>Skyline</h1>
  <p class="sub">Type any city to detect its current weather.</p>
  <div class="search">
    <input id="city" placeholder="e.g. Kanpur, Tokyo, Lisbon" autocomplete="off">
    <button id="go">Detect</button>
  </div>
  <div class="card" id="card">
    <p class="empty">Enter a city name above and press Detect to pull live conditions.</p>
  </div>
  <footer>Live data via Open-Meteo · no key required</footer>
</div>
 
<script>
const codeMap = {
  0:"Clear sky",1:"Mainly clear",2:"Partly cloudy",3:"Overcast",
  45:"Fog",48:"Depositing rime fog",
  51:"Light drizzle",53:"Drizzle",55:"Dense drizzle",
  61:"Light rain",63:"Rain",65:"Heavy rain",
  71:"Light snow",73:"Snow",75:"Heavy snow",
  80:"Rain showers",81:"Rain showers",82:"Violent rain showers",
  95:"Thunderstorm",96:"Thunderstorm with hail",99:"Thunderstorm with hail"
};
 
const card = document.getElementById('card');
const input = document.getElementById('city');
const btn = document.getElementById('go');
 
async function detect(){
  const city = input.value.trim();
  if(!city){ return; }
  btn.disabled = true;
  card.innerHTML = '<p class="loading">Detecting…</p>';
  try{
    const geoRes = await fetch(`https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(city)}&count=1`);
    const geo = await geoRes.json();
    if(!geo.results || !geo.results.length){
      card.innerHTML = '<p class="error">No place found with that name. Try a different spelling or add a country.</p>';
      return;
    }
    const p = geo.results[0];
    const wRes = await fetch(`https://api.open-meteo.com/v1/forecast?latitude=${p.latitude}&longitude=${p.longitude}&current=temperature_2m,relative_humidity_2m,apparent_temperature,weather_code,wind_speed_10m&timezone=auto`);
    const w = await wRes.json();
    const c = w.current;
    const cond = codeMap[c.weather_code] || "Unknown conditions";
    card.innerHTML = `
      <p class="place">${p.name}${p.admin1 ? ', '+p.admin1 : ''}, ${p.country}</p>
      <div class="temp">${Math.round(c.temperature_2m)}°C</div>
      <p class="cond">${cond}</p>
      <div class="grid">
        <div class="stat"><b>${Math.round(c.apparent_temperature)}°C</b><span>Feels like</span></div>
        <div class="stat"><b>${c.relative_humidity_2m}%</b><span>Humidity</span></div>
        <div class="stat"><b>${Math.round(c.wind_speed_10m)} km/h</b><span>Wind</span></div>
      </div>`;
  }catch(err){
    card.innerHTML = '<p class="error">Couldn\'t reach the weather service right now. If this page is hosted, try downloading it and opening it in your own browser instead.</p>';
  }finally{
    btn.disabled = false;
  }
}
 
btn.addEventListener('click', detect);
input.addEventListener('keydown', e => { if(e.key === 'Enter') detect(); });
</script>
</body>
</html>
