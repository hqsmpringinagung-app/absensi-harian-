<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Absensi Siswa Digital - SMP HQ</title>

    <!-- Libraries -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.25/jspdf.plugin.autotable.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap" rel="stylesheet">

    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Poppins', sans-serif; }
        body { background: linear-gradient(135deg, #1e40af, #3b82f6, #10b981); min-height: 100vh; padding: 20px; }
        
        /* LOGIN STYLES */
        #login-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: linear-gradient(135deg, #1e40af, #10b981);
            display: flex; align-items: center; justify-content: center; z-index: 9999;
        }
        .login-card {
            background: white; padding: 40px; border-radius: 24px; width: 100%; max-width: 400px;
            text-align: center; box-shadow: 0 15px 35px rgba(0,0,0,0.2);
        }
        .login-card h2 { color: #1e3a8a; margin-bottom: 10px; }
        .login-card p { color: #64748b; font-size: 14px; margin-bottom: 25px; }
        .login-input { width: 100%; padding: 12px; margin-bottom: 15px; border: 1.5px solid #cbd5e1; border-radius: 10px; outline: none; }
        .btn-login { width: 100%; background: #2563eb; color: white; border: none; padding: 12px; border-radius: 10px; font-weight: 600; cursor: pointer; }
        
        /* MAIN APP CONTENT (Hidden by default) */
        #main-app { display: none; }

        .container { max-width: 1100px; margin: auto; background: #fff; border-radius: 24px; padding: 30px; box-shadow: 0 20px 40px rgba(0,0,0,0.2); }
        .header { text-align: center; margin-bottom: 30px; }
        .logo { width: 60px; height: 60px; background: #2563eb; color: white; display: flex; align-items: center; justify-content: center; font-size: 24px; font-weight: bold; border-radius: 15px; margin: 0 auto 10px; }
        .header h1 { color: #1e3a8a; font-size: 26px; }
        
        .form-box { background: #f8fafc; padding: 20px; border-radius: 18px; margin-bottom: 30px; border: 1px solid #e2e8f0; }
        .form-grid { display: grid; grid-template-columns: 2fr 1fr 1.5fr 1fr 1fr; gap: 15px; align-items: flex-end; }
        .input-group { display: flex; flex-direction: column; gap: 8px; }
        .input-group label { font-size: 11px; font-weight: 700; color: #64748b; text-transform: uppercase; }
        
        input, select { width: 100%; padding: 12px; border: 1.5px solid #cbd5e1; border-radius: 10px; font-size: 14px; outline: none; transition: 0.3s; }
        .tanggal-box { display: flex; gap: 4px; }
        .tanggal-box input { padding: 12px 5px; text-align: center; }

        .btn { border: none; border-radius: 10px; padding: 12px; font-weight: 600; cursor: pointer; transition: 0.2s; }
        .btn-primary { background: #2563eb; color: white; height: 48px; }
        .btn-success { background: #10b981; color: white; }
        .btn-danger { background: #ef4444; color: white; padding: 5px 10px; font-size: 11px; border-radius: 6px; }

        .rekap { display: grid; grid-template-columns: repeat(4, 1fr); gap: 15px; margin-bottom: 30px; }
        .card { padding: 15px; border-radius: 15px; color: white; text-align: center; }
        .bg-hadir { background: #10b981; } .bg-izin { background: #f59e0b; }
        .bg-sakit { background: #3b82f6; } .bg-alfa { background: #ef4444; }

        .table-container { overflow-x: auto; background: white; border-radius: 15px; border: 1px solid #e2e8f0; }
        table { width: 100%; border-collapse: collapse; min-width: 800px; }
        th { background: #f1f5f9; color: #475569; padding: 15px; text-align: left; font-size: 13px; }
        td { padding: 12px 15px; font-size: 14px; border-bottom: 1px solid #f1f5f9; }
        
        .day-separator { background: #e2e8f0 !important; font-weight: 700; color: #1e3a8a; text-align: center; font-size: 12px; letter-spacing: 1px; }
        .status-pill { padding: 4px 10px; border-radius: 20px; font-size: 11px; font-weight: 600; color: white; }

        @media (max-width: 992px) { .form-grid { grid-template-columns: 1fr 1fr; } .rekap { grid-template-columns: 1fr 1fr; } }
        @media (max-width: 600px) { .form-grid { grid-template-columns: 1fr; } }
    </style>
</head>
<body>

<!-- TAMPILAN LOGIN -->
<div id="login-overlay">
    <div class="login-card">
        <div class="logo">HQ</div>
        <h2>Siswa Digital Login</h2>
        <p>Silahkan masuk untuk mengelola absensi</p>
        <input type="text" id="username" class="login-input" placeholder="Username (admin)">
        <input type="password" id="password" class="login-input" placeholder="Password (admin123)">
        <button class="btn-login" onclick="cekLogin()">Masuk Ke Sistem</button>
        <p id="pesan-error" style="color: #ef4444; margin-top: 15px; display: none; font-size: 12px;">Username atau Password salah!</p>
    </div>
</div>

<!-- TAMPILAN UTAMA APLIKASI -->
<div id="main-app">
    <div class="container">
        <div class="header">
            <div style="display: flex; justify-content: flex-end;">
                <button onclick="logout()" style="background:none; border:none; color:#ef4444; cursor:pointer; font-weight:bold; font-size:12px;">Keluar Sistem X</button>
            </div>
            <div class="logo">HQ</div>
            <h1>Absensi Siswa Digital</h1>
            <p>SMP HAMALATUL QURAN RINGINAGUNG</p>
        </div>

        <div class="form-box">
            <div class="form-grid">
                <div class="input-group">
                    <label>Nama Siswa</label>
                    <input type="text" id="nama" placeholder="Nama lengkap...">
                </div>
                <div class="input-group">
                    <label>Kelas</label>
                    <select id="kelas">
                        <option value="">Pilih...</option>
                        <option>Kelas 7</option>
                        <option>Kelas 8</option>
                        <option>Kelas 9</option>
                    </select>
                </div>
                <div class="input-group">
                    <label>Tanggal</label>
                    <div class="tanggal-box">
                        <input type="number" id="tgl" placeholder="DD">
                        <input type="number" id="bln" placeholder="MM">
                        <input type="number" id="thn" placeholder="YYYY">
                    </div>
                </div>
                <div class="input-group">
                    <label>Status</label>
                    <select id="status">
                        <option value="Hadir">Hadir</option>
                        <option value="Izin">Izin</option>
                        <option value="Sakit">Sakit</option>
                        <option value="Alfa">Alfa</option>
                    </select>
                </div>
                <button class="btn btn-primary" onclick="tambahData()">SIMPAN DATA</button>
            </div>
        </div>

        <div class="rekap">
            <div class="card bg-hadir"><h4>Hadir</h4><h2 id="countHadir">0</h2></div>
            <div class="card bg-izin"><h4>Izin</h4><h2 id="countIzin">0</h2></div>
            <div class="card bg-sakit"><h4>Sakit</h4><h2 id="countSakit">0</h2></div>
            <div class="card bg-alfa"><h4>Alfa</h4><h2 id="countAlfa">0</h2></div>
        </div>

        <div class="table-container">
            <table id="tableAbsensi">
                <thead>
                    <tr>
                        <th>No</th>
                        <th>Nama Siswa</th>
                        <th>Kelas</th>
                        <th>Hari</th>
                        <th>Tanggal</th>
                        <th>Status</th>
                        <th>Aksi</th>
                    </tr>
                </thead>
                <tbody id="tbody"></tbody>
            </table>
        </div>

        <div style="margin-top: 25px; display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 15px;">
            <h3>Total: <span id="total">0</span> Data</h3>
            <div>
                <button class="btn btn-danger" onclick="resetSemuaData()" style="margin-right: 10px;">Reset Data</button>
                <button class="btn btn-success" onclick="downloadPDF()">Download PDF</button>
            </div>
        </div>
    </div>
</div>

<script>
    // --- LOGIKA LOGIN ---
    function cekLogin() {
        const user = document.getElementById("username").value;
        const pass = document.getElementById("password").value;

        // Default login: admin / admin123
        if (user === "admin" && pass === "admin123") {
            sessionStorage.setItem("isLoggedIn", "true");
            tampilkanApp();
        } else {
            document.getElementById("pesan-error").style.display = "block";
        }
    }

    function tampilkanApp() {
        if (sessionStorage.getItem("isLoggedIn") === "true") {
            document.getElementById("login-overlay").style.display = "none";
            document.getElementById("main-app").style.display = "block";
            renderTable(); // Jalankan render setelah login berhasil
        }
    }

    function logout() {
        sessionStorage.removeItem("isLoggedIn");
        location.reload();
    }

    // --- LOGIKA ABSENSI ---
    let dataAbsensi = JSON.parse(localStorage.getItem("absensi_hq")) || [];

    window.onload = () => {
        // Cek apakah sudah login sebelumnya di sesi ini
        if (sessionStorage.getItem("isLoggedIn") === "true") {
            tampilkanApp();
        }

        const d = new Date();
        document.getElementById('tgl').value = d.getDate();
        document.getElementById('bln').value = d.getMonth() + 1;
        document.getElementById('thn').value = d.getFullYear();
    };

    function tambahData() {
        const nama = document.getElementById("nama").value;
        const kelas = document.getElementById("kelas").value;
        const tgl = document.getElementById("tgl").value;
        const bln = document.getElementById("bln").value;
        const thn = document.getElementById("thn").value;
        const status = document.getElementById("status").value;

        if (!nama || !kelas || !tgl || !bln || !thn) {
            alert("Isi semua data!"); return;
        }

        const dateObj = new Date(`${thn}-${bln}-${tgl}`);
        const hariList = ["Minggu", "Senin", "Selasa", "Rabu", "Kamis", "Jumat", "Sabtu"];
        const hari = hariList[dateObj.getDay()];
        const formatTgl = `${tgl.toString().padStart(2,'0')}-${bln.toString().padStart(2,'0')}-${thn}`;

        const item = { id: Date.now(), nama, kelas, hari, formatTgl, status };
        dataAbsensi.push(item);
        
        localStorage.setItem("absensi_hq", JSON.stringify(dataAbsensi));
        document.getElementById("nama").value = "";
        renderTable();
    }

    function renderTable() {
        const tbody = document.getElementById("tbody");
        if(!tbody) return; // Guard jika belum di-render
        tbody.innerHTML = "";
        
        let counts = { Hadir: 0, Izin: 0, Sakit: 0, Alfa: 0 };
        let lastDate = "";

        // Sort data berdasarkan tanggal untuk memastikan pemisah bekerja benar
        dataAbsensi.sort((a,b) => new Date(a.id) - new Date(b.id));

        dataAbsensi.forEach((item, index) => {
            if (lastDate !== item.formatTgl) {
                const sepRow = tbody.insertRow();
                const cell = sepRow.insertCell(0);
                cell.colSpan = 7;
                cell.className = "day-separator";
                cell.innerText = `--- DATA TANGGAL: ${item.formatTgl} ---`;
                lastDate = item.formatTgl;
            }

            const row = tbody.insertRow();
            row.innerHTML = `
                <td>${index + 1}</td>
                <td style="font-weight:600">${item.nama}</td>
                <td>${item.kelas}</td>
                <td>${item.hari}</td>
                <td>${item.formatTgl}</td>
                <td><span class="status-pill bg-${item.status.toLowerCase()}">${item.status}</span></td>
                <td><button class="btn-danger" onclick="hapusData(${item.id})">Hapus</button></td>
            `;
            counts[item.status]++;
        });

        document.getElementById("countHadir").innerText = counts.Hadir;
        document.getElementById("countIzin").innerText = counts.Izin;
        document.getElementById("countSakit").innerText = counts.Sakit;
        document.getElementById("countAlfa").innerText = counts.Alfa;
        document.getElementById("total").innerText = dataAbsensi.length;
    }

    function hapusData(id) {
        if (confirm("Hapus data ini?")) {
            dataAbsensi = dataAbsensi.filter(x => x.id !== id);
            localStorage.setItem("absensi_hq", JSON.stringify(dataAbsensi));
            renderTable();
        }
    }

    function resetSemuaData() {
        if (confirm("Hapus semua data absensi selamanya?")) {
            dataAbsensi = [];
            localStorage.removeItem("absensi_hq");
            renderTable();
        }
    }

    function downloadPDF() {
        const { jsPDF } = window.jspdf;
        const doc = new jsPDF();
        
        doc.setFontSize(16);
        doc.text("LAPORAN ABSENSI SMP HAMALATUL QURAN", 14, 20);
        doc.setFontSize(10);
        doc.text("Ringinagung - Magetan | Tanggal Cetak: " + new Date().toLocaleDateString('id-ID'), 14, 28);

        const rows = [];
        let lastDate = "";

        dataAbsensi.forEach((item, index) => {
            if (lastDate !== item.formatTgl) {
                rows.push([{ content: `TANGGAL: ${item.formatTgl}`, colSpan: 6, styles: { fillColor: [226, 232, 240], fontStyle: 'bold', halign: 'center' } }]);
                lastDate = item.formatTgl;
            }
            rows.push([index + 1, item.nama, item.kelas, item.hari, item.formatTgl, item.status]);
        });

        doc.autoTable({
            startY: 35,
            head: [['No', 'Nama Siswa', 'Kelas', 'Hari', 'Tanggal', 'Status']],
            body: rows,
            theme: 'grid',
            headStyles: { fillColor: [37, 99, 235] },
            margin: { top: 35 }
        });

        doc.save(`Absensi_HQ_${new Date().getTime()}.pdf`);
    }
</script>

</body>
</html>
