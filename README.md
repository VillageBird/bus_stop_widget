# bus_stop_widget


<!DOCTYPE html>
<html lang="el">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Στάση Περίπτερο</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  html, body { width: 100%; height: 100%; overflow: hidden; background: #0f172a; color: #e2e8f0; font-family: Arial, Helvetica, sans-serif; }
  .screen { width: 100%; height: 100%; padding: 3vh 3vw; }
  h1 { text-align: center; font-size: 4vh; margin-bottom: 2vh; }
  .widget { background: #1e293b; border-radius: 1.5vh; padding: 2vh 2vw; border-top: 0.8vh solid #facc15; width: 70%; margin: 0 auto; }
  .widget h2 { font-size: 3.4vh; margin-bottom: 1.5vh; }
  .head, .arrival { display: -webkit-flex; display: flex; text-align: center; padding: 1.2vh 0; font-size: 3vh; }
  .head span, .arrival span { -webkit-flex: 1; flex: 1; }
  .head { color: #94a3b8; font-size: 2.2vh; border-bottom: 2px solid #334155; }
  .arrival { border-bottom: 1px solid #334155; }
  .route { font-weight: bold; color: #facc15; }
  .veh { color: #94a3b8; }
  .time { font-weight: bold; }
  .time.soon { color: #4ade80; }
  .msg { color: #94a3b8; padding: 1.4vh 0; font-size: 2.8vh; text-align: center; }
  #updated { text-align: center; font-size: 2vh; color: #64748b; margin-top: 2vh; }
  #debug { text-align: center; font-size: 1.6vh; color: #475569; margin-top: 1vh; }
</style>
</head>
<body>
  <div class="screen">
    <h1>Επόμενα λεωφορεία</h1>
    <div class="widget">
      <h2>Περίπτερο</h2>
      <div class="head"><span>Route code</span><span>Veh code</span><span>Λεπτά</span></div>
      <div id="rows"></div>
    </div>
    <div id="updated"></div>
    <div id="debug"></div>
  </div>

<script>
  // ---- ΡΥΘΜΙΣΕΙΣ ----
  var STOP_CODE = "750025";
  var REFRESH_SECONDS = 30;
  var URL_BASE = "https://telematics.oasa.gr/api/?act=getStopArrivals&p1=";
  var SHOW_DEBUG = true; // βάλε false όταν δουλέψει
  // ---------------------

  var rows = document.getElementById("rows");
  var updated = document.getElementById("updated");
  var debug = document.getElementById("debug");

  function el(tag, cls, text) {
    var e = document.createElement(tag);
    if (cls) e.className = cls;
    if (text !== undefined) e.textContent = text;
    return e;
  }

  function showMessage(t) {
    rows.innerHTML = "";
    rows.appendChild(el("div", "msg", t));
  }

  function render(data) {
    var list = Object.prototype.toString.call(data) === "[object Array]" ? data : [];
    if (list.length === 0) { showMessage("Δεν υπάρχουν αφίξεις"); return; }
    list.sort(function (a, b) { return Number(a.btime2) - Number(b.btime2); });
    rows.innerHTML = "";
    for (var i = 0; i < list.length; i++) {
      var a = list[i];
      var mins = Number(a.btime2);
      var row = el("div", "arrival");
      row.appendChild(el("span", "route", String(a.route_code)));
      row.appendChild(el("span", "veh", String(a.veh_code)));
      row.appendChild(el("span", "time" + (mins <= 3 ? " soon" : ""), mins + " λ."));
      rows.appendChild(row);
    }
  }

  function done(msg) {
    var d = new Date();
    var hh = ("0" + d.getHours()).slice(-2);
    var mm = ("0" + d.getMinutes()).slice(-2);
    updated.textContent = "Τελευταία ενημέρωση: " + hh + ":" + mm;
    if (SHOW_DEBUG) debug.textContent = (msg ? msg + " | " : "") + navigator.userAgent;
  }

  function refresh() {
    var xhr = new XMLHttpRequest();
    xhr.open("GET", URL_BASE + STOP_CODE, true);
    xhr.timeout = 10000;
    xhr.onload = function () {
      if (xhr.status !== 200) {
        showMessage("Μη διαθέσιμο: HTTP " + xhr.status);
        done("HTTP " + xhr.status);
        return;
      }
      try {
        render(JSON.parse(xhr.responseText));
        done("");
      } catch (e) {
        showMessage("Μη διαθέσιμο: μη έγκυρη απάντηση");
        done("JSON σφάλμα: " + e.message);
      }
    };
    xhr.onerror = function () {
      showMessage("Μη διαθέσιμο: σφάλμα δικτύου ή CORS");
      done("xhr error");
    };
    xhr.ontimeout = function () {
      showMessage("Μη διαθέσιμο: timeout");
      done("timeout");
    };
    xhr.send();
  }

  refresh();
  setInterval(refresh, REFRESH_SECONDS * 1000);
</script>
</body>
</html>