<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="utf-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1"/>
<title>Kalkulator Multiple Taruhan 2D — Final (Per-Pola)</title>
<style>
  body { font-family: Arial, sans-serif; margin: 20px; background: #f9f9f9; color: #333; }
  h2,h3 { color:#0077cc; margin-bottom:8px; }
  select, input, textarea, button { font-size:14px; }
  select, input, textarea { padding:6px; border-radius:6px; border:1px solid #aaa; }
  textarea { resize:vertical; width:100%; }
  .container { background:#fff; padding:14px; border-radius:10px; box-shadow:0 2px 6px rgba(0,0,0,0.08); margin-bottom:16px; }
  .controls-row { display:flex; gap:8px; align-items:center; flex-wrap:wrap; margin-top:8px; }
  button { padding:8px 12px; border-radius:6px; border:1px solid #bbb; background:#f5f5f5; cursor:pointer; }
  button.primary { background:#0077cc; color:white; border-color:#0062a3; }
  button.success { background:#009933; color:white; border-color:#077a1a; }
  button.danger { background:#cc0000; color:white; border-color:#a30000; }
  .small { font-size:13px; color:#555; }
  table { width:100%; border-collapse:collapse; margin-top:10px; font-size:13px; }
  th, td { border:1px solid #ddd; padding:8px; text-align:center; }
  th { background:#e8f6ff; }
  .pola-row { display:flex; gap:6px; flex-wrap:wrap; margin-bottom:8px; }
  .pola-btn { padding:6px 10px; border-radius:6px; border:1px solid #bbb; background:#f5f5f5; cursor:pointer; font-weight:600; }
  .pola-btn.active { background:#0077cc; color:white; border-color:#0062a3; }
  #statusHariIni { color:#009933; font-weight:700; margin-top:10px; }
</style>
</head>
<body>

<h2>Kalkulator Multiple Taruhan 2D — Multi Pasaran (Per-Pola)</h2>

<div class="container">
  <div class="small">Pilih Pasaran:</div>
  <select id="pasaranSelect" onchange="gantiPasaran()">
    <option>HAWAI</option>
    <option>TOKYO</option>
    <option>PAPUA</option>
    <option>MEXICO</option>
    <option>HOCIMINH</option>
    <option>CAMBODIA</option>
    <option>BUSAN</option>
    <option>SIDNEY</option>
    <option>SIDNEY LOTTO</option>
    <option>HANOI</option>
    <option>CHINA</option>
    <option>KOREA</option>
    <option>JAPAN</option>
    <option>SINGAPORE</option>
    <option>KAMBOJA</option>
    <option>JEJU</option>
    <option>PCSO</option>
    <option>PENANG</option>
    <option>TAIWAN</option>
    <option>HONGKONG</option>
    <option>HONGKONG LOTTO</option>
  </select>
</div>

<!-- POLA A..T -->
<div class="container">
  <div class="small">Pola (A → T) — pilih 1 pola aktif (data terpisah per pola)</div>
  <div class="pola-row" id="polaRow"></div>
  <div class="small">Pola aktif berfungsi sebagai konteks untuk semua aksi (Hitung / Menang / Kalah / Undo / Reset pola).</div>
</div>

<div class="container">
  <div style="display:flex;gap:12px;flex-wrap:wrap;align-items:flex-end">
    <div>
      <div class="small">Jumlah nomor dipasang (M):</div>
      <input type="number" id="jumlahNomor" value="10" min="1" max="99">
    </div>
    <div>
      <div class="small">Payout ratio:</div>
      <input type="number" id="payout" value="100">
    </div>
    <div>
      <div class="small">Target keuntungan per menang:</div>
      <input type="number" id="target" value="50000">
    </div>
    <div>
      <div class="small">Minimal taruhan per nomor:</div>
      <input type="number" id="minTaruhan" value="100" step="100">
    </div>
  </div>

  <p class="small">Total kekalahan (pola aktif): <input id="kerugian" value="0" readonly style="border:none;background:transparent;"></p>
  <p class="small">Total modal kumulatif (pola aktif): <input id="totalModal" value="0" readonly style="border:none;background:transparent;"></p>

  <div class="controls-row" id="mainControls">
    <button class="primary" onclick="hitung()">Hitung Multiple</button>
    <button class="success" onclick="tambahMenangPola()">Tambah Menang (pola aktif)</button>
    <button class="success" onclick="tambahKalahPola()">Tambah Kekalahan (pola aktif)</button>
    <button onclick="batalkanHariTerakhirPola()">Batalkan Hari Terakhir (pola aktif)</button>
    <button class="danger" onclick="resetPolaIni()">Reset Pola Ini</button>
    <button onclick="simulasi()">Simulasi 10 Hari</button>
    <button onclick="exportData()">Ekspor Data</button>
    <button onclick="triggerImport()">Impor Data</button>
    <input type="file" id="importFile" accept=".json" style="display:none" onchange="importData(event)">
  </div>

  <p id="statusHariIni"></p>
</div>

<div class="container" id="hasil"></div>

<!-- Tabel rekap semua pola -->
<div class="container">
  <h3>Rekap Semua Pola (Pola A → T)</h3>
  <table id="tabelRekapPola">
    <tr><th>Pola</th><th>Total Kekalahan</th><th>Total Modal</th><th>Tanggal Update</th></tr>
  </table>
</div>

<!-- Histori (Pola Aktif) -->
<div class="container">
  <h3>Histori (Pola Aktif)</h3>
  <table id="tabelHistori">
    <tr><th>Hari</th><th>Tanggal</th><th>Status</th><th>Modal</th><th>Total Kekalahan</th><th>Total Modal</th></tr>
  </table>
  <div class="summary" id="ringkasan"></div>
</div>

<!-- Textarea catatan per pasaran -->
<div class="container">
  <h3>Catatan Keluaran (per pasaran)</h3>
  <textarea id="catatanPasaran" rows="8"></textarea>
  <div style="margin-top:8px">
    <button class="primary" onclick="simpanCatatan()">Simpan</button>
    <button class="danger" onclick="hapusCatatan()">Hapus</button>
    <button class="success" onclick="salinCatatan()">Salin</button>
  </div>
  <p class="small" style="margin-top:8px">Catatan disimpan per pasaran dan akan dimunculkan otomatis saat mengganti pasaran.</p>
</div>

<script>
// ---- DATA & INIT ----
const POLA_LIST = Array.from({length:20}, (_,i) => String.fromCharCode(65+i)); // A..T
let dataSemuaPasaran = JSON.parse(localStorage.getItem("dataMultiPasaran") || "{}");
let pasaranAktif = localStorage.getItem("lastActivePasaran") || document.getElementById("pasaranSelect").value;
let polaAktif = localStorage.getItem("polaAktif") || "A";

// pastikan semua struktur data ada
function ensureAllPasaranExist() {
  const sel = document.getElementById("pasaranSelect");
  for (let i = 0; i < sel.options.length; i++) {
    const name = sel.options[i].text;
    if (!dataSemuaPasaran[name]) {
      dataSemuaPasaran[name] = { catatan: "", polas:{} };
      for (const p of POLA_LIST) dataSemuaPasaran[name].polas[p]={ totalKalah:0,totalModal:0,histori:[],modalTerakhir:0,lastActionDate:"" };
    } else {
      if (!dataSemuaPasaran[name].polas) dataSemuaPasaran[name].polas={};
      for(const p of POLA_LIST){
        if(!dataSemuaPasaran[name].polas[p]) dataSemuaPasaran[name].polas[p]={ totalKalah:0,totalModal:0,histori:[],modalTerakhir:0,lastActionDate:"" };
      }
      if(typeof dataSemuaPasaran[name].catatan==="undefined") dataSemuaPasaran[name].catatan="";
    }
  }
}
ensureAllPasaranExist();

function simpanData(){
  localStorage.setItem("dataMultiPasaran", JSON.stringify(dataSemuaPasaran));
  localStorage.setItem("lastActivePasaran", pasaranAktif);
  localStorage.setItem("polaAktif", polaAktif);
}

// ---- POLA BUTTONS ----
function renderPolaButtons(){
  const row = document.getElementById("polaRow");
  row.innerHTML="";
  for(const p of POLA_LIST){
    const btn=document.createElement("button");
    btn.className="pola-btn"+(p===polaAktif?" active":"");
    btn.innerText=p;
    btn.onclick=()=>{pilihPola(p);};
    row.appendChild(btn);
  }
}
function pilihPola(p){ polaAktif=p; simpanData(); renderPolaButtons(); tampilkanData(); }

// ---- HELPER ----
function tanggalLokalSekarang(){ return new Date().toLocaleDateString("id-ID"); }
function hariIniISO(){ const d=new Date(); return d.toISOString().split("T")[0]; }
function updateStatusHariIni(text){ document.getElementById("statusHariIni").innerText = text ? `📅 Hari ini sudah dicatat: ${text}` : ""; }

// ---- HITUNG ----
function hitung(){
  const f=parseInt(document.getElementById("payout").value,10);
  const T=parseInt(document.getElementById("target").value,10);
  const M=parseInt(document.getElementById("jumlahNomor").value,10);
  const minBet=parseInt(document.getElementById("minTaruhan").value,10);
  if(isNaN(f)||isNaN(T)||isNaN(M)||isNaN(minBet)){ alert("Masukkan nilai numerik yang valid."); return;}
  if(M>=f){ alert("Jumlah nomor tidak boleh ≥ payout ratio!"); return; }
  const dPas=dataSemuaPasaran[pasaranAktif];
  const polaObj=dPas.polas[polaAktif];
  const L=polaObj.totalKalah||0;
  let s=(T+L)/(f-M);
  s=Math.ceil(s/minBet)*minBet;
  const totalModal=M*s;
  const totalMenang=f*s;
  const net=totalMenang-totalModal;
  polaObj.modalTerakhir=totalModal;
  dPas.polas[polaAktif]=polaObj;
  dataSemuaPasaran[pasaranAktif]=dPas;
  simpanData();
  document.getElementById("hasil").innerHTML=`
    <p>Stake ideal per nomor: <strong>Rp${s.toLocaleString()}</strong></p>
    <p>Total modal hari ini (pola aktif): <strong>Rp${totalModal.toLocaleString()}</strong></p>
    <p>Total menang (1 nomor kena): <strong>Rp${totalMenang.toLocaleString()}</strong></p>
    <p><strong>Net hasil:</strong> Rp${net.toLocaleString()}</p>
  `;
  tampilkanData();
}

// ---- MENANG / KALAH / UNDO / RESET ----
function tambahMenangPola(){
  const dPas=dataSemuaPasaran[pasaranAktif]; const polaObj=dPas.polas[polaAktif];
  const todayISO=hariIniISO();
  if(polaObj.lastActionDate===todayISO) return alert("Pola ini sudah dicatat hari ini!");
  if(!confirm("Yakin ingin menandai pola ini sebagai MENANG hari ini?")) return;
  if(!polaObj.modalTerakhir || polaObj.modalTerakhir===0) return alert("Hitung dulu (tekan Hitung Multiple) sebelum mencatat kemenangan!");
  const f=parseInt(document.getElementById("payout").value,10);
  const M=parseInt(document.getElementById("jumlahNomor").value,10);
  const s=polaObj.modalTerakhir/M;
  const totalMenang=f*s; const net=totalMenang-polaObj.modalTerakhir;
  polaObj.totalKalah=Math.max(0,(polaObj.totalKalah||0)-net);
  polaObj.totalModal=(polaObj.totalModal||0)+polaObj.modalTerakhir;
  polaObj.lastActionDate=todayISO;
  polaObj.histori.push({ hari:polaObj.histori.length+1, tanggal:tanggalLokalSekarang(), status:"Menang", modal:polaObj.modalTerakhir, totalKalah:polaObj.totalKalah, totalModal:polaObj.totalModal });
  dPas.polas[polaAktif]=polaObj; dataSemuaPasaran[pasaranAktif]=dPas; simpanData(); tampilkanData();
  updateStatusHariIni("MENANG");
}
function tambahKalahPola(){
  const dPas=dataSemuaPasaran[pasaranAktif]; const polaObj=dPas.polas[polaAktif];
  const todayISO=hariIniISO();
  if(polaObj.lastActionDate===todayISO) return alert("Pola ini sudah dicatat hari ini!");
  if(!confirm("Yakin ingin menandai pola ini sebagai KALAH hari ini?")) return;
  if(!polaObj.modalTerakhir || polaObj.modalTerakhir===0) return alert("Hitung dulu sebelum menambah kekalahan!");
  polaObj.totalKalah=(polaObj.totalKalah||0)+polaObj.modalTerakhir;
  polaObj.totalModal=(polaObj.totalModal||0)+polaObj.modalTerakhir;
  polaObj.lastActionDate=todayISO;
  polaObj.histori.push({ hari:polaObj.histori.length+1, tanggal:tanggalLokalSekarang(), status:"Kalah", modal:polaObj.modalTerakhir, totalKalah:polaObj.totalKalah, totalModal:polaObj.totalModal });
  dPas.polas[polaAktif]=polaObj; dataSemuaPasaran[pasaranAktif]=dPas; simpanData(); tampilkanData();
  updateStatusHariIni("KALAH");
}
function batalkanHariTerakhirPola(){
  const dPas=dataSemuaPasaran[pasaranAktif]; const polaObj=dPas.polas[polaAktif];
  if(!polaObj.histori || polaObj.histori.length===0) return alert("Belum ada histori untuk dibatalkan.");
  if(!confirm("Batalkan entri hari terakhir untuk pola ini?")) return;
  polaObj.histori.pop();
  if(polaObj.histori.length===0){ polaObj.totalKalah=0; polaObj.totalModal=0; polaObj.lastActionDate=""; }
  else{
    const last=polaObj.histori[polaObj.histori.length-1]; polaObj.totalKalah=last.totalKalah||0; polaObj.totalModal=last.totalModal||0;
    const parts=(last.tanggal||"").split('/'); if(parts.length===3) polaObj.lastActionDate=`${parts[2]}-${parts[1].padStart(2,'0')}-${parts[0].padStart(2,'0')}`; else polaObj.lastActionDate="";
  }
  dPas.polas[polaAktif]=polaObj; dataSemuaPasaran[pasaranAktif]=dPas; simpanData(); tampilkanData(); alert("Entri hari terakhir berhasil dibatalkan.");
}
function resetPolaIni(){
  if(!confirm("Reset data hanya untuk pola aktif?")) return;
  const dPas=dataSemuaPasaran[pasaranAktif];
  dPas.polas[polaAktif]={ totalKalah:0,totalModal:0,histori:[],modalTerakhir:0,lastActionDate:"" };
  dataSemuaPasaran[pasaranAktif]=dPas; simpanData(); tampilkanData(); alert("Pola "+polaAktif+" telah direset.");
}

// ---- TAMPILKAN DATA ----
function tampilkanData(){
  const dPas=dataSemuaPasaran[pasaranAktif]; const polaObj=dPas.polas[polaAktif];
  document.getElementById("kerugian").value=polaObj.totalKalah||0;
  document.getElementById("totalModal").value=polaObj.totalModal||0;

  // tabel histori pola aktif
  const tableHist=document.getElementById("tabelHistori"); tableHist.innerHTML="<tr><th>Hari</th><th>Tanggal</th><th>Status</th><th>Modal</th><th>Total Kekalahan</th><th>Total Modal</th></tr>";
  (polaObj.histori||[]).forEach(r=>{
    const tr=document.createElement("tr"); tr.innerHTML=`<td>${r.hari}</td><td>${r.tanggal||""}</td><td>${r.status}</td><td>Rp${r.modal.toLocaleString()}</td><td>Rp${r.totalKalah.toLocaleString()}</td><td>Rp${r.totalModal.toLocaleString()}</td>`; tableHist.appendChild(tr);
  });

  // rekap semua pola
  const tableRekap=document.getElementById("tabelRekapPola"); tableRekap.innerHTML="<tr><th>Pola</th><th>Total Kekalahan</th><th>Total Modal</th><th>Tanggal Update</th></tr>";
  POLA_LIST.forEach(p=>{
    const po=polaAktif===p?polaObj:dPas.polas[p];
    const tr=document.createElement("tr");
    tr.innerHTML=`<td>${p}</td><td>Rp${po.totalKalah.toLocaleString()}</td><td>Rp${po.totalModal.toLocaleString()}</td><td>${po.lastActionDate||""}</td>`;
    tableRekap.appendChild(tr);
  });

  // catatan pasaran
  document.getElementById("catatanPasaran").value=dPas.catatan||"";
}

// ---- CATATAN PASARAN ----
function simpanCatatan(){ const d=dataSemuaPasaran[pasaranAktif]; d.catatan=document.getElementById("catatanPasaran").value; dataSemuaPasaran[pasaranAktif]=d; simpanData(); alert("Catatan disimpan."); }
function hapusCatatan(){ if(confirm("Hapus catatan pasaran ini?")){ dataSemuaPasaran[pasaranAktif].catatan=""; simpanData(); document.getElementById("catatanPasaran").value=""; alert("Catatan dihapus."); } }
function salinCatatan(){ document.getElementById("catatanPasaran").select(); document.execCommand("copy"); alert("Catatan disalin ke clipboard."); }

// ---- PASARAN ----
function gantiPasaran(){
  pasaranAktif=document.getElementById("pasaranSelect").value;
  simpanData(); tampilkanData();
}

// ---- IMPORT / EXPORT ----
function exportData(){ const blob=new Blob([JSON.stringify(dataSemuaPasaran,null,2)],{type:"application/json"}); const a=document.createElement("a"); a.href=URL.createObjectURL(blob); a.download="dataMultiPasaran.json"; a.click(); }
function triggerImport(){ document.getElementById("importFile").click(); }
function importData(e){ const file=e.target.files[0]; if(!file) return; const reader=new FileReader(); reader.onload=function(evt){ try{ const obj=JSON.parse(evt.target.result); dataSemuaPasaran=obj; ensureAllPasaranExist(); simpanData(); tampilkanData(); alert("Data berhasil diimpor."); }catch(err){ alert("File tidak valid!"); } }; reader.readAsText(file); }

// ---- INIT ----
document.getElementById("pasaranSelect").value=pasaranAktif;
renderPolaButtons();
tampilkanData();
</script>
</body>
</html>
