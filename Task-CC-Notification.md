# 🔔 Tugas Praktikum: Sistem Peringatan Suhu (AWS SNS)

## 💡 Apa itu AWS SNS?
**AWS SNS (Simple Notification Service)** adalah layanan pesan otomatis. Di tugas ini, kita akan membuat sistem monitoring suhu yang **otomatis mengirim email peringatan** jika suhu ruangan terlalu panas (> 30 derajat Celcius).

---

## 🎯 Tujuan Tugas
Murid dapat mengintegrasikan logika pemrosesan data (Lambda) dengan layanan notifikasi (SNS).

---

## 🚀 Langkah-Langkah Praktikum

### 1. Membuat Topic SNS
1. Buka dashboard **SNS** -> klik **"Topics"** di menu kiri -> **"Create topic"**.
2. **Type**: Pilih **Standard**.
3. **Name**: `AlertSuhuRuangan`. Klik **Create topic**.
4. Setelah dibuat, klik nama topik yang baru dibuat.
5. Klik tab **"Subscriptions"** -> **"Create subscription"**.
6. **Protocol**: Pilih **"Email"**.
7. **Endpoint**: Masukkan alamat email Anda.
8. Klik **"Create subscription"**.
9. **PENTING**: Buka email Anda, cari pesan dari AWS, dan klik **"Confirm subscription"**.

### 2. Mengambil ARN Topik
1. Di halaman detail Topik Anda, cari baris **"Topic ARN"**.
2. Salin teks tersebut (formatnya: `arn:aws:sns:region:akun:AlertSuhuRuangan`). **Simpan ini!**

### 3. Update Kode Lambda
1. Buka fungsi Lambda Anda -> tab **"Code"**.
2. Hapus semua kode dan masukkan kode di bawah ini:
   *(Jangan lupa ganti `ISI_ARN_TOPIC_KAMU_DISINI` dengan ARN yang Anda salin tadi)*

```python
import json
import boto3

sns = boto3.client('sns')
TOPIC_ARN = 'ISI_ARN_TOPIC_KAMU_DISINI'

def lambda_handler(event, context):
    params = event.get('queryStringParameters', {})
    
    # 1. Bagian API: Jika ada input suhu, cek dan kirim email
    if 'suhu' in params:
        suhu = int(params.get('suhu'))
        if suhu > 30:
            sns.publish(
                TopicArn=TOPIC_ARN,
                Message=f"⚠️ PERINGATAN: Suhu ruangan mencapai {suhu} derajat!",
                Subject="Alert: Suhu Terlalu Panas!"
            )
            return {'statusCode': 200, 'body': f"Suhu {suhu}C - Peringatan terkirim!"}
        return {'statusCode': 200, 'body': f"Suhu {suhu}C - Aman."}

    # 2. Bagian UI: Jika tidak ada input, kirim UI Kalkulator
    return {
        'statusCode': 200,
        'headers': {"Content-Type": "text/html"},
        'body': """
        <!DOCTYPE html>
        <html>
        <body style="font-family:sans-serif; text-align:center; padding:50px;">
            <h2>Sistem Monitoring Suhu</h2>
            <input type="number" id="suhu" placeholder="Masukkan Suhu (e.g 35)">
            <button onclick="cekSuhu()">Cek Suhu</button>
            <h3 id="hasil">Status: -</h3>
            <script>
                async function cekSuhu() {
                    const s = document.getElementById('suhu').value;
                    const url = window.location.href + "?suhu=" + s;
                    const res = await fetch(url).then(r => r.text());
                    document.getElementById('hasil').innerText = res;
                }
            </script>
        </body>
        </html>
        """
    }
```
3. Klik tombol **"Deploy"** (biru).

### 4. Mengetes Hasil
1. Buka kembali **Function URL** Anda di browser.
2. Masukkan angka `25` -> klik **"Cek Suhu"** (Hasil: Aman).
3. Masukkan angka `35` -> klik **"Cek Suhu"** (Hasil: Peringatan terkirim!).
4. Cek inbox email Anda, pesan peringatan akan masuk!
