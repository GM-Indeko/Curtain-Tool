<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Curtain Tool – Demo</title>
<style>
  :root { --gap: 12px; --radius: 10px; }
  body { font-family: system-ui, -apple-system, Segoe UI, Roboto, sans-serif; margin: 0; padding: 16px; background:#f7f7f9; color:#1a1a1a; }
  header { margin-bottom: 14px; }
  h1 { font-size: 20px; margin: 0 0 4px; }
  .card { background:#fff; border-radius: var(--radius); padding:16px; box-shadow: 0 6px 18px rgba(0,0,0,0.06); }
  .grid { display:grid; grid-template-columns: 1fr; gap: var(--gap); }
  @media (min-width:720px){ .grid { grid-template-columns: repeat(2,1fr); } }
  label { font-size: 13px; color:#444; display:block; margin-bottom:6px; }
  select, input[type="text"], textarea {
    width:100%; padding:10px 12px; border:1px solid #d9d9df; border-radius:8px; background:#fff; font-size:15px;
  }
  select:disabled { background:#f0f0f4; color:#777; }
  .row { display:flex; gap: var(--gap); align-items:center; }
  .pill { display:inline-block; padding:4px 8px; font-size:12px; border-radius:999px; background:#eef3ff; color:#2b55d9; }
  .muted { color:#666; font-size:13px; }
  .result { font-size:15px; line-height:1.5; }
  .mono { font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; }
  footer { margin-top: 18px; font-size: 12px; color:#666; }
  details { background:#fafafa; border:1px dashed #ddd; border-radius:8px; padding:10px 12px; }
  summary { cursor:pointer; font-weight:600; }
</style>
</head>
<body>
  <header>
    <h1>Curtain Tool — Flow Demo <span class="pill">Fixing → Position → Track → Heading → Hook</span></h1>
    <div class="muted">Mobile-friendly demo for sales flow. Paste your data (optional) under “Load full JSON” below.</div>
  </header>

  <div class="card grid">
    <div>
      <label>Fixing</label>
      <select id="fixing"></select>
    </div>
    <div>
      <label>Curtain Position</label>
      <select id="position"></select>
    </div>
    <div>
      <label>Track</label>
      <select id="track"></select>
    </div>
    <div>
      <label>Heading</label>
      <select id="heading"></select>
    </div>
    <div>
      <label>Hook / Pin</label>
      <select id="hook"></select>
    </div>
    <div>
      <label>Offset vs Track Top (mm)</label>
      <input id="offset" type="text" readonly />
    </div>
  </div>

  <div class="card" style="margin-top:14px">
    <div class="result">
      <div><strong>Derived From</strong>: <span id="derived" class="mono">—</span></div>
      <div class="muted">Shown for Gather Top / Middle / Bottom (maps to 2071/Pin10, 2072/Pin25, 2074/Pin50).</div>
    </div>
  </div>

  <div class="card" style="margin-top:14px">
    <details>
      <summary>Load full JSON (optional)</summary>
      <div class="muted" style="margin:10px 0 6px">
        Paste the contents of your <span class="mono">curtain_app_matrix_nested.json</span> here, then click “Use JSON below”.
      </div>
      <textarea id="jsonInput" rows="10" placeholder='{"Face-fix":{"Front":{...}}}'></textarea>
      <div class="row" style="margin-top:10px">
        <button id="loadBtn">Use JSON below</button>
        <button id="resetBtn">Reset to demo data</button>
      </div>
    </details>
  </div>

  <footer>
    Tip: On iPad, you can host this via GitHub Pages or just open the file directly in Files → long-press → Share → Open in your browser.
  </footer>

<script>
/*** --- DEMO DATA (works out of the box) --- ***/
/* Replace DEMO_DATA with your full nested JSON by pasting in the box above and clicking “Use JSON below”. */
let MATRIX = {
  "Face-fix": {
    "Front": {
      "S74 Venice": {
        "Pleat": [
          {"Hook/Pin":"2074","Offset vs Track Top (mm":"+11","Default":"Default lined","Derived From":""},
          {"Hook/Pin":"Pin 50","Offset vs Track Top (mm":"+12","Default":"Default sheer","Derived From":""},
          {"Hook/Pin":"2073","Offset vs Track Top (mm":"−1","Default":"","Derived From":""},
          {"Hook/Pin":"2072","Offset vs Track Top (mm":"−14","Default":"","Derived From":""},
          {"Hook/Pin":"2071","Offset vs Track Top (mm":"−32","Default":"","Derived From":""},
          {"Hook/Pin":"Pin 35","Offset vs Track Top (mm":"−3","Default":"","Derived From":""},
          {"Hook/Pin":"Pin 25","Offset vs Track Top (mm":"−13","Default":"","Derived From":""},
          {"Hook/Pin":"Pin 10","Offset vs Track Top (mm":"−28","Default":"","Derived From":""}
        ],
        "Wave": [
          {"Hook/Pin":"Wave tape","Offset vs Track Top (mm":"−31","Default":"Always underslung","Derived From":""}
        ],
        "Gather Top": [
          {"Hook/Pin":"Pin 10","Offset vs Track Top (mm":"−28","Default":"","Derived From":"2071/Pin 10 equivalent"}
        ],
        "Gather Middle": [
          {"Hook/Pin":"Pin 25","Offset vs Track Top (mm":"−13","Default":"","Derived From":"2072/Pin 25 equivalent"}
        ]
      }
    },
    "Rear": {
      "S74 Venice": {
        "Pleat": [
          {"Hook/Pin":"2072","Offset vs Track Top (mm":"−14","Default":"","Derived From":""},
          {"Hook/Pin":"2071","Offset vs Track Top (mm":"−32","Default":"","Derived From":""},
          {"Hook/Pin":"Pin 25","Offset vs Track Top (mm":"−13","Default":"","Derived From":""},
          {"Hook/Pin":"Pin 10","Offset vs Track Top (mm":"−28","Default":"","Derived From":""}
        ],
        "Wave": [
          {"Hook/Pin":"Wave tape","Offset vs Track Top (mm":"−31","Default":"Always underslung","Derived From":""}
        ],
        "Gather Top": [
          {"Hook/Pin":"Pin 10","Offset vs Track Top (mm":"−28","Default":"","Derived From":"2071/Pin 10 equivalent"}
        ],
        "Gather Middle": [
          {"Hook/Pin":"Pin 25","Offset vs Track Top (mm":"−13","Default":"","Derived From":"2072/Pin 25 equivalent"}
        ]
      }
    }
  },
  "Top-fix": {
    "Either": {
      "S50 Milan": {
        "Pleat": [
          {"Hook/Pin":"2072","Offset vs Track Top (mm":"−4","Default":"Default lined","Derived From":""},
          {"Hook/Pin":"Pin 25","Offset vs Track Top (mm":"−3","Default":"Default sheer","Derived From":""},
          {"Hook/Pin":"2071","Offset vs Track Top (mm":"−22","Default":"","Derived From":""},
          {"Hook/Pin":"Pin 10","Offset vs Track Top (mm":"−18","Default":"","Derived From":""}
        ],
        "Wave": [
          {"Hook/Pin":"Wave tape","Offset vs Track Top (mm":"−24","Default":"Always underslung","Derived From":""}
        ],
        "Gather Top": [
          {"Hook/Pin":"Pin 10","Offset vs Track Top (mm":"−18","Default":"","Derived From":"2071/Pin 10 equivalent"}
        ],
        "Gather Middle": [
          {"Hook/Pin":"Pin 25","Offset vs Track Top (mm":"−3","Default":"","Derived From":"2072/Pin 25 equivalent"}
        ]
      }
    }
  }
};

/*** --- UI logic --- ***/
const $ = sel => document.querySelector(sel);
const fixing = $("#fixing"), position = $("#position"), track = $("#track"),
      heading = $("#heading"), hook = $("#hook"), offset = $("#offset"), derived = $("#derived");

function fill(sel, items, placeholder="— Select —") {
  sel.innerHTML = "";
  const opt0 = document.createElement("option");
  opt0.value = ""; opt0.textContent = placeholder; sel.appendChild(opt0);
  items.forEach(v => { const o=document.createElement("option"); o.value=o.textContent=v; sel.appendChild(o); });
  sel.disabled = items.length === 0;
}

function onFixingChange() {
  const fx = fixing.value;
  const positions = fx && MATRIX[fx] ? Object.keys(MATRIX[fx]) : [];
  fill(position, positions, "— Position —");
  track.innerHTML = heading.innerHTML = hook.innerHTML = "";
  offset.value = ""; derived.textContent = "—";
}
function onPositionChange() {
  const fx = fixing.value, pos = position.value;
  const tracks = fx && pos && MATRIX[fx] && MATRIX[fx][pos] ? Object.keys(MATRIX[fx][pos]) : [];
  fill(track, tracks, "— Track —");
  heading.innerHTML = hook.innerHTML = ""; offset.value = ""; derived.textContent = "—";
}
function onTrackChange() {
  const fx = fixing.value, pos = position.value, tr = track.value;
  const headings = fx && pos && tr && MATRIX[fx][pos][tr] ? Object.keys(MATRIX[fx][pos][tr]) : [];
  fill(heading, headings, "— Heading —");
  hook.innerHTML = ""; offset.value = ""; derived.textContent = "—";
}
function onHeadingChange() {
  const fx = fixing.value, pos = position.value, tr = track.value, hd = heading.value;
  const rows = fx && pos && tr && hd && MATRIX[fx][pos][tr][hd] ? MATRIX[fx][pos][tr][hd] : [];
  fill(hook, rows.map(r => r["Hook/Pin"]), "— Hook/Pin —");
  offset.value = ""; derived.textContent = "—";
}
function onHookChange() {
  const fx = fixing.value, pos = position.value, tr = track.value, hd = heading.value, hk = hook.value;
  const rows = (MATRIX?.[fx]?.[pos]?.[tr]?.[hd] ?? []).filter(r => r["Hook/Pin"] === hk);
  const row = rows[0] || null;
  offset.value = row ? row["Offset vs Track Top (mm)"] : "";
  derived.textContent = row ? (row["Derived From"] || "—") : "—";
}

/*** Wire up ***/
fixing.addEventListener("change", onFixingChange);
position.addEventListener("change", onPositionChange);
track.addEventListener("change", onTrackChange);
heading.addEventListener("change", onHeadingChange);
hook.addEventListener("change", onHookChange);

/*** Initialize ***/
fill(fixing, Object.keys(MATRIX), "— Fixing —");

/*** Load full JSON from textarea ***/
document.getElementById("loadBtn").addEventListener("click", () => {
  const txt = document.getElementById("jsonInput").value.trim();
  if (!txt) { alert("Paste JSON first."); return; }
  try {
    MATRIX = JSON.parse(txt);
    fill(fixing, Object.keys(MATRIX), "— Fixing —");
    position.innerHTML = track.innerHTML = heading.innerHTML = hook.innerHTML = "";
    offset.value = ""; derived.textContent = "—";
    alert("Loaded custom JSON.");
  } catch (e) {
    alert("Invalid JSON.\n" + e.message);
  }
});
document.getElementById("resetBtn").addEventListener("click", () => {
  location.reload();
});
</script>
</body>
    </html>