# 🔔 Tugas Praktikum: Sistem Peringatan Suhu (AWS SNS)

## 💡 Studi Kasus: Smart Room Monitor
Bayangkan kamu sedang membangun sistem monitoring suhu di sebuah ruang server. Kita ingin sistem **otomatis mengirim email peringatan** jika suhu ruangan terlalu panas (> 30 derajat Celcius). Kita akan menggunakan **AWS Lambda** sebagai sensor pengolah data dan **AWS SNS** sebagai pengirim notifikasi.

---

## 🎯 Tujuan Tugas
Murid dapat mengintegrasikan logika *if-else* (pemrosesan data) dengan layanan notifikasi (SNS) untuk membuat sistem *alerting* otomatis.

---

## 🚀 Langkah-Langkah Praktikum

### 1. Membuat Topic SNS
1. Buka dashboard **SNS** -> **Topics** -> **Create topic**.
2. **Type**: Pilih **Standard**.
3. **Name**: `AlertSuhuRuangan-NamaSiswa`.
4. Klik **Create topic**.
5. Buka *Topic* -> **Subscriptions** -> **Create subscription**.
6. **Protocol**: **Email**.
7. **Endpoint**: Masukkan email kamu. Klik **Create subscription**.
8. **Cek Email**: Konfirmasi di inbox email kamu.

### 2. Update Kode Lambda
1. Buka fungsi Lambda kamu.
2. Ganti kodenya dengan logika deteksi suhu di bawah ini:

```python
import json
import boto3

sns = boto3.client('sns')
TOPIC_ARN = 'ISI_ARN_TOPIC_KAMU_DISINI'

def lambda_handler(event, context):
    params = event.get('queryStringParameters', {})
    suhu = int(params.get('suhu', 25)) # Default 25 derajat
    
    if suhu > 30:
        # Kirim notifikasi jika terlalu panas
        sns.publish(
            TopicArn=TOPIC_ARN,
            Message=f"⚠️ PERINGATAN: Suhu ruangan mencapai {suhu} derajat Celcius!",
            Subject="Alert: Suhu Terlalu Panas!"
        )
        return {'statusCode': 200, 'body': f"Suhu {suhu}C - Peringatan Terkirim!"}
    
    return {'statusCode': 200, 'body': f"Suhu {suhu}C - Aman."}
```

### 3. Tes Sistem
1. Buka **Function URL** Lambda Anda di browser.
2. Tambahkan parameter `?suhu=25` di akhir URL (Hasil: Aman).
3. Tambahkan parameter `?suhu=35` di akhir URL (Hasil: Peringatan terkirim ke email!).

---

## ⚠️ PENTING: Pembersihan
**WAJIB** hapus *Topic* SNS dan fungsi Lambda agar tidak terkena biaya!
