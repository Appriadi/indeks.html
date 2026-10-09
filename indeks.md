<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Aplikasi Pemindai Barcode</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
    body { background-color: #f4f6f9; padding: 20px; display: flex; justify-content: center; align-items: center; min-height: 100vh; }
    .card { background: #ffffff; padding: 24px; border-radius: 12px; max-width: 480px; width: 100%; box-shadow: 0 4px 12px rgba(0,0,0,0.1); }
    h2 { text-align: center; margin-bottom: 20px; color: #1e293b; }
    .btn { width: 100%; padding: 12px 20px; font-size: 1rem; border: none; border-radius: 8px; cursor: pointer; font-weight: 600; display: inline-flex; align-items: center; justify-content: center; gap: 8px; }
    .btn-primary { background: #2563eb; color: #ffffff; }
    .btn-danger { background: #dc2626; color: #ffffff; margin-top: 12px; }
    #reader { width: 100%; margin-top: 16px; border-radius: 8px; overflow: hidden; background: #000000; }
    .status { margin-top: 16px; padding: 12px; border-radius: 8px; display: none; font-size: 0.9rem; word-break: break-word; }
    .status-success { background: #dcfce7; color: #15803d; border: 1px solid #bbf7d0; }
    .status-error { background: #fee2e2; color: #b91c1c; border: 1px solid #fecaca; }
    .status-info { background: #dbeafe; color: #1e40af; border: 1px solid #bfdbfe; }
  </style>

  <!-- Library Pemindai Barcode HTML5 -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html5-qrcode/2.3.8/html5-qrcode.min.js"></script>
</head>
<body>

  <div class="card">
    <h2>Pemindaian Barcode</h2>

    <button id="btn-start" class="btn btn-primary" onclick="startCamera()">📷 Scan Barcode</button>
    <button id="btn-stop" class="btn btn-danger" onclick="stopCamera()" style="display: none;">🛑 Hentikan Kamera</button>

    <div id="reader" style="display: none;"></div>
    <div id="status" class="status"></div>
  </div>

  <script>
    // TEMPELKAN URL WEB APP APPS SCRIPT KAMU DI SINI:
    const WEB_APP_URL = "https://script.google.com/macros/s/AKfycbyBpdp1ATFmgWpGTj1_00nO9-NDAM3QlRmH5uiEGTy_LbahlaT9oqTATZe6v77GeLTc/exec";

    let html5QrCode = null;
    let isProcessing = false;

    function startCamera() {
      showStatus("Meminta akses kamera...", "info");
      document.getElementById('reader').style.display = 'block';

      if (!html5QrCode) {
        html5QrCode = new Html5Qrcode("reader");
      }

      const config = { fps: 10, qrbox: { width: 250, height: 150 } };

      html5QrCode.start(
        { facingMode: "environment" },
        config,
        onScanSuccess,
        onScanError
      ).then(() => {
        document.getElementById('btn-start').style.display = 'none';
        document.getElementById('btn-stop').style.display = 'inline-flex';
        showStatus("Kamera aktif. Arahkan ke barcode.", "info");
      }).catch(err => {
        // Fallback jika facingMode environment tidak tersedia
        html5QrCode.start({ facingMode: "user" }, config, onScanSuccess, onScanError)
          .then(() => {
            document.getElementById('btn-start').style.display = 'none';
            document.getElementById('btn-stop').style.display = 'inline-flex';
            showStatus("Kamera aktif. Arahkan ke barcode.", "info");
          })
          .catch(error => {
            showStatus("Gagal membuka kamera: " + error, "error");
            document.getElementById('reader').style.display = 'none';
          });
      });
    }

    function onScanSuccess(decodedText) {
      if (isProcessing) return;
      isProcessing = true;

      showStatus(`Barcode Terbaca: <b>${decodedText}</b>. Menyimpan ke Google Sheets...`, "info");
      stopCamera();

      // Mengirim data ke Google Apps Script via API fetch
      fetch(WEB_APP_URL, {
        method: "POST",
        mode: "no-cors",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ barcode: decodedText })
      })
      .then(() => {
        showStatus(`Berhasil! Data <b>${decodedText}</b> tersimpan di Google Sheets.`, "success");
        isProcessing = false;
      })
      .catch(err => {
        showStatus("Gagal mengirim data ke server: " + err, "error");
        isProcessing = false;
      });
    }

    function onScanError(err) {
      // Proses scanning sedang berjalan
    }

    function stopCamera() {
      if (html5QrCode && html5QrCode.isScanning) {
        html5QrCode.stop().then(() => {
          html5QrCode.clear();
          document.getElementById('reader').style.display = 'none';
          document.getElementById('btn-start').style.display = 'inline-flex';
          document.getElementById('btn-stop').style.display = 'none';
        }).catch(err => console.error(err));
      }
    }

    function showStatus(msg, type) {
      const el = document.getElementById('status');
      el.innerHTML = msg;
      el.className = 'status status-' + type;
      el.style.display = 'block';
    }
  </script>
</body>
</html>
