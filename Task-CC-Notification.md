# 🔔 Tugas Praktikum: Sistem Peringatan Suhu (AWS SNS)

## 🎯 Tujuan Tugas
Membuat sistem peringatan otomatis menggunakan **AWS Lambda** (sebagai pemroses data) dan **AWS SNS** (sebagai pengirim notifikasi email).

---

## 🚀 Langkah 1: Membuat Fungsi Lambda dari Awal

1. **Buka AWS Console**: Ketik "Lambda" di kolom pencarian.
2. **Klik "Create function"** (tombol oranye di kanan atas).
3. **Pilih opsi**: "Author from scratch".
4. **Isi konfigurasi**:
   - **Function name**: `FunctionAlert-NamaSiswa`
   - **Runtime**: Pilih **Python 3.12**.
   - **Architecture**: Biarkan default (x86_64).
5. **Role (PENTING)**:
   - Klik "Change default execution role".
   - Pilih "Create a new role with basic Lambda permissions".
   - Klik **"Create function"**.

---

## 🚀 Langkah 2: Membuat Topik SNS (Notifikasi)
1. Buka dashboard **SNS** -> **Topics** -> **Create topic**.
2. **Type**: **Standard**. **Name**: `AlertSuhuRuangan-NamaSiswa`. Klik **Create topic**.
3. Buka topik tersebut -> tab **"Subscriptions"** -> **"Create subscription"**.
4. **Protocol**: **Email**. **Endpoint**: Masukkan email kamu. Klik **Create subscription**.
5. **Konfirmasi**: Cek inbox email kamu, cari pesan dari AWS, klik **"Confirm subscription"**.

---

## 🚀 Langkah 3: Memberikan Izin Lambda ke SNS
Agar Lambda bisa mengirim email ke SNS:
1. Kembali ke halaman fungsi Lambda Anda -> tab **"Configuration"** -> **"Permissions"**.
2. Klik link **Role name** (warna biru). Anda akan diarahkan ke halaman IAM.
3. Klik **"Add permissions"** -> **"Attach policies"**.
4. Cari `AmazonSNSFullAccess`, centang, dan klik **"Attach policies"**.

---

## 🚀 Langkah 4: Memasukkan Kode Program
1. Di halaman fungsi Lambda, tab **"Code"**, hapus isi `lambda_function.py`.
2. Tempel kode di bawah ini. **Ganti** `ISI_ARN_TOPIC_KAMU_DISINI` dengan ARN Topik Anda (bisa dilihat di halaman detail Topic SNS).
3. Klik tombol **"Deploy"** (biru).

```python
import json
import boto3

sns = boto3.client('sns')
TOPIC_ARN = 'ISI_ARN_TOPIC_KAMU_DISINI'

def lambda_handler(event, context):
    params = event.get('queryStringParameters', {})
    if 'suhu' in params:
        suhu = int(params.get('suhu'))
        if suhu > 30:
            sns.publish(
                TopicArn=TOPIC_ARN,
                Message=f"⚠️ PERINGATAN: Suhu mencapai {suhu}C!",
                Subject="Alert: Suhu Terlalu Panas!"
            )
            return {'statusCode': 200, 'body': f"Suhu {suhu}C - Peringatan terkirim!"}
        return {'statusCode': 200, 'body': f"Suhu {suhu}C - Aman."}

    # UI Kalkulator
    return {'statusCode': 200, 'headers': {"Content-Type": "text/html"}, 'body': """
    <html><body style="text-align:center; padding:50px;">
        <h2>Monitoring Suhu</h2>
        <input type="number" id="suhu" placeholder="Angka Suhu">
        <button onclick="f()">Cek</button>
        <h3 id="hasil">Status: -</h3>
        <script>async function f(){
            const s=document.getElementById('suhu').value;
            const res=await fetch(window.location.href+'?suhu='+s).then(r=>r.text());
            document.getElementById('hasil').innerText=res;
        }</script>
    </body></html>"""}
```

---

## 🚀 Langkah 5: Mengaktifkan Function URL
1. Tab **"Configuration"** -> **"Function URL"**.
2. Klik **"Create function URL"**.
3. **Auth type**: **NONE**.
4. **CORS**: Klik **Edit**, centang **"Allow all origins (*)"**, klik **Save**.
5. Salin URL yang muncul dan buka di browser!

---

## 🚀 Langkah 6: Pengujian (Testing)
1. Buka URL yang telah disalin di browser Anda.
2. **Uji Kondisi Aman**: Masukkan angka `25` pada input suhu, lalu klik tombol **"Cek"**. Layar akan menampilkan "Status: Suhu 25C - Aman."
3. **Uji Kondisi Bahaya**: Masukkan angka `35` pada input suhu, lalu klik tombol **"Cek"**. Layar akan menampilkan "Status: Suhu 35C - Peringatan terkirim!".
4. **Cek Email**: Pastikan Anda menerima email notifikasi dari AWS (cek folder *Spam* jika tidak ditemukan di *Inbox*).

---

## ⚠️ PENTING: Pembersihan (Wajib!)
Setelah selesai, **WAJIB** hapus sumber daya agar tidak ada biaya:
1. **SNS**: Hapus Topik dan Subscription yang Anda buat.
2. **Lambda**: Pilih fungsi -> **Actions** -> **Delete function**.
",file_path:
