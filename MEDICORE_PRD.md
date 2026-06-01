# MediCore HMS — Product Requirements Document
**Version:** 1.0  
**Date:** 1 Juni 2026  
**Author:** Hendy Unggul  
**Status:** Ready for Development — Demo Sprint

---

## 1. VISION

**MediCore** adalah Hospital Management System berbasis PWA yang mengintegrasikan seluruh alur pelayanan pasien — dari registrasi mandiri, antrian real-time, rekam medis digital, hingga integrasi BPJS — dalam satu platform yang bisa diakses dari tablet, PC, dan smartphone tanpa instalasi.

### Tagline
> "Dari loket ke dokter — tanpa kertas, tanpa antre panjang, tanpa salah informasi."

### Tujuan Demo
Menampilkan seluruh alur kerja nyata dengan dummy data — dokter, suster, dan pasien semua aktif — sebelum kontrak. Setelah kontrak, develop full scale dengan penyesuaian fitur.

---

## 2. PLATFORM & ARSITEKTUR

### PWA — Satu Codebase, Semua Device
```
Dokter    → Tablet Android / iPad
Suster    → PC atau Tablet  
Pasien    → HP Android (mayoritas Indonesia)
Admin     → PC / Laptop
```
Tidak perlu App Store. Install via browser → "Add to Home Screen". Update otomatis.

### Database Architecture
```
PRIMARY:   PostgreSQL Cloud (Supabase)
           Real-time subscriptions
           Row Level Security per role
           Auto-backup harian

BACKUP:    IndexedDB local (Dexie.js)
           Offline queue saat internet mati
           Auto-sync saat koneksi kembali
           Cache data kritikal per role

UPTIME:    99.9% cloud = max 8.7 jam downtime/tahun
           Dengan local backup = zero disruption
```

### Tech Stack
| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 14 + Tailwind CSS |
| Real-time | Supabase Realtime |
| Database | Supabase PostgreSQL |
| Local backup | Dexie.js (IndexedDB) |
| Auth | Supabase Auth (role-based) |
| PWA | next-pwa + Service Worker |
| OCR | Google Cloud Vision API |
| Voice input | Web Speech API (browser native) |
| QR Code | qrcode.react + html5-qrcode |
| Export | jspdf + xlsx |
| Hosting | Vercel (frontend) + Supabase (backend) |
| Notifikasi | Web Push + WhatsApp Business API |

---

## 3. USER ROLES & ACCESS

### Role Matrix
```
                    DOKTER  SUSTER  ADMIN   PASIEN
Rekam medis          RW      R       R       R*
Diagnosis            RW      —       R       R*
Resep                RW      R       R       R*
Hasil lab/penunjang  RW      R       R       R*
Antrian              R       RW      RW      R
Jadwal dokter        R       RW      RW      —
Vital signs          R       RW      R       R*
Billing              R       R       RW      R*
User management      —       —       RW      —
Dashboard analytics  R       —       RW      —
Reschedule/cancel    —       RW      RW      RW*

R = Read  W = Write  R* = data milik pasien sendiri
```

### Device per Role
```
DOKTER — Tablet:
• Touch-first, semua elemen besar
• Voice input untuk anamnesis
• Font minimum 16px
• Swipe antar pasien
• Calm, clinical, tidak distraktif

SUSTER — PC atau Tablet:
• Keyboard-friendly
• Tab navigation
• Tabel dense tapi readable
• Form validation helpful
• Multi-window friendly (PC)

PASIEN — Android HP:
• Super simple — satu aksi per layar
• Zero login kompleks — via QR token
• Loading cepat, data minimal
• Nomor antrian besar di tengah

ADMIN — PC:
• Dashboard penuh
• Export semua data
• User management
• Analytics & laporan
```

---

## 4. DATA MODEL

```sql
-- USERS & AUTH
users
  id              uuid PK
  name            varchar
  role            enum(doctor, nurse, admin, patient)
  email           varchar UNIQUE nullable
  phone           varchar UNIQUE
  is_active       boolean default true
  created_at      timestamp

-- DOCTORS
doctors
  id              uuid PK
  user_id         FK → users
  specialization  varchar
  schedule        jsonb     -- jam praktik per hari
  room            varchar
  tariff_general  integer   -- tarif umum
  bio             text nullable

-- PATIENTS — Anchor Data
patients
  id                    uuid PK
  nik                   varchar(16) UNIQUE NOT NULL
  bpjs_number           varchar(13) nullable
  emergency_phone       varchar(15) NOT NULL
  full_name             varchar     -- untuk dokumen resmi
  nickname              varchar     -- untuk komunikasi
  date_of_birth         date        -- auto-derive dari NIK
  gender                enum(M,F)   -- auto-derive dari NIK
  blood_type            varchar nullable
  insurance_type        enum(bpjs, bpjs_rujukan, general)
  bpjs_class            enum(1,2,3) nullable
  bpjs_status           enum(active, inactive) nullable
  bpjs_faskes           varchar nullable
  registration_status   enum(pending, verified, complete)
  self_registered       boolean default false
  reg_token             varchar nullable
  reg_token_expires     timestamp nullable
  first_visit           date
  created_at            timestamp
  updated_at            timestamp

-- HEALTH ANCHORS (Layer penting — selalu visible)
patient_allergies
  id              uuid PK
  patient_id      FK → patients
  allergen_name   varchar
  allergen_type   enum(drug, food, environment, chemical)
  drug_class      varchar nullable
  reaction_type   enum(mild, moderate, severe, anaphylaxis)
  notes           text nullable
  reported_by     enum(patient, nurse, doctor)
  verified_by     FK → users nullable
  is_active       boolean default true
  created_at      timestamp

patient_conditions
  id              uuid PK
  patient_id      FK → patients
  condition_name  varchar
  icd10_code      varchar nullable
  onset_date      date nullable
  is_chronic      boolean default true
  notes           text nullable
  reported_by     enum(patient, nurse, doctor)
  verified_by     FK → users nullable
  is_active       boolean default true
  created_at      timestamp

patient_medications_ongoing
  id              uuid PK
  patient_id      FK → patients
  drug_name       varchar
  dosage          varchar nullable
  frequency       varchar nullable
  prescribed_by   varchar nullable
  notes           text nullable
  reported_by     enum(patient, nurse, doctor)
  is_active       boolean default true
  created_at      timestamp

-- PATIENT DOCUMENTS
patient_documents
  id              uuid PK
  patient_id      FK → patients
  doc_type        enum(ktp, bpjs, kis, other)
  file_url        varchar
  ocr_result      jsonb
  ocr_confidence  decimal
  verified_by     FK → users nullable
  verified_at     timestamp nullable
  upload_method   enum(self_upload, staff_scan, manual)
  created_at      timestamp

-- SLOTS & QUEUE
slots
  id              uuid PK
  doctor_id       FK → doctors
  date            date
  start_time      time
  end_time        time
  max_patients    integer default 20
  booked_count    integer default 0
  is_active       boolean
  day_of_week     enum(MON,TUE,WED,THU,FRI,SAT,SUN)

queues
  id                uuid PK
  patient_id        FK → patients
  doctor_id         FK → doctors
  slot_id           FK → slots
  date              date
  queue_number      varchar    -- A-001, B-002 per poli
  queue_type        enum(bpjs, bpjs_rujukan, general)
  status            enum(registered, waiting, called,
                         in_progress, done, cancelled,
                         no_show, rescheduled)
  chief_complaint   text
  called_at         timestamp nullable
  started_at        timestamp nullable
  done_at           timestamp nullable
  created_at        timestamp

-- RESCHEDULE & CANCEL (Blindspot 1 — BARU)
queue_changes
  id              uuid PK
  queue_id        FK → queues
  change_type     enum(reschedule, cancel)
  reason          text nullable
  old_slot_id     FK → slots
  new_slot_id     FK → slots nullable
  requested_by    enum(patient, nurse, admin, doctor)
  requested_by_id uuid
  policy_applied  varchar    -- "free_cancel", "admin_fee"
  fee_charged     integer default 0
  status          enum(pending, approved, done)
  created_at      timestamp

-- VITAL SIGNS
vital_signs
  id              uuid PK
  queue_id        FK → queues
  patient_id      FK → patients
  blood_pressure  varchar    -- "120/80"
  temperature     decimal
  pulse           integer
  spo2            integer
  weight          decimal
  height          decimal
  bmi             decimal    -- auto-calculate
  gds             integer nullable
  recorded_by     FK → users
  recorded_at     timestamp

-- MEDICAL RECORDS
medical_records
  id              uuid PK
  queue_id        FK → queues
  patient_id      FK → patients
  doctor_id       FK → doctors
  anamnesis       text
  physical_exam   text
  diagnosis       text
  icd10_code      varchar nullable
  therapy_plan    text
  notes           text nullable
  follow_up_date  date nullable
  created_at      timestamp

-- PRESCRIPTIONS
prescriptions
  id                uuid PK
  medical_record_id FK → medical_records
  drug_name         varchar
  dosage            varchar
  frequency         varchar
  duration_days     integer
  instructions      text nullable
  status            enum(pending, prepared, dispensed)

-- LAB & PENUNJANG (Blindspot 2 — BARU)
lab_orders
  id                uuid PK
  medical_record_id FK → medical_records
  patient_id        FK → patients
  doctor_id         FK → doctors
  order_type        enum(lab, radiology, ecg, other)
  order_name        varchar
  clinical_notes    text nullable
  status            enum(ordered, processing, done)
  ordered_at        timestamp
  done_at           timestamp nullable

lab_results
  id              uuid PK
  lab_order_id    FK → lab_orders
  patient_id      FK → patients
  result_data     jsonb      -- flexible per test type
  result_file_url varchar nullable  -- PDF/image hasil
  normal_range    jsonb nullable
  interpretation  text nullable    -- catatan laboran
  doctor_notes    text nullable    -- catatan dokter
  is_critical     boolean default false
  patient_notified boolean default false
  created_at      timestamp

-- REFERRALS
referrals
  id                uuid PK
  medical_record_id FK → medical_records
  referred_to       varchar
  reason            text
  created_at        timestamp

-- BPJS INTEGRATION
sep_records
  id              uuid PK
  queue_id        FK → queues
  patient_id      FK → patients
  sep_number      varchar
  bpjs_number     varchar
  referral_number varchar nullable
  visit_type      enum(outpatient, inpatient, emergency)
  poli            varchar
  doctor_id       FK → doctors
  diagnosis_icd10 varchar nullable
  created_at      timestamp
  status          enum(active, closed, claimed)

bpjs_coverage_rules
  id              uuid PK
  icd10_code      varchar
  icd10_name      varchar
  service_type    varchar
  service_name    varchar
  is_covered      boolean
  coverage_notes  text
  max_frequency   varchar nullable
  requires_referral boolean
  updated_at      timestamp

bpjs_formularium
  id              uuid PK
  drug_name       varchar
  generic_name    varchar
  drug_class      varchar
  is_covered      boolean
  coverage_level  enum(DPHO, non_DPHO)
  restriction     text nullable
  updated_at      timestamp

-- MEDICATION REMINDERS
medication_reminders
  id              uuid PK
  prescription_id FK → prescriptions
  patient_id      FK → patients
  reminder_times  time[]     -- array jam pengingat
  is_active       boolean default true
  created_at      timestamp

medication_logs
  id              uuid PK
  prescription_id FK → prescriptions
  patient_id      FK → patients
  scheduled_at    timestamp
  taken_at        timestamp nullable
  status          enum(taken, missed, late)
  tip_shown       text nullable
  tip_opened      boolean default false

-- HEALTH CONTENT
health_contents
  id              uuid PK
  content_type    enum(tip, myth_buster, insight, reminder_text)
  title           varchar
  body            text
  condition_tags  text[]
  drug_tags       text[]
  time_of_day     enum(morning, evening, any)
  treatment_phase enum(acute_early, acute_mid, acute_late, chronic, any)
  min_days_gap    integer default 2
  is_active       boolean
  reviewed_at     timestamp
  created_at      timestamp

content_delivery_log
  id              uuid PK
  patient_id      FK → patients
  content_id      FK → health_contents
  delivered_at    timestamp
  opened          boolean default false
  opened_at       timestamp nullable

-- NOTIFICATIONS
notifications
  id              uuid PK
  recipient_type  enum(doctor, nurse, patient, admin)
  recipient_id    uuid
  type            varchar    -- 20 types (lihat section 8)
  channel         enum(push, wa, sms)
  payload         jsonb
  sent_at         timestamp nullable
  status          enum(pending, sent, failed)
  queue_id        FK → queues nullable
```

---

## 5. PATIENT APP — 7 SCREENS MVP

### Screen 1: Onboarding & Identifikasi
```
Flow:
1. Upload foto KTP + BPJS (OCR auto-extract)
2. Pilih tipe: BPJS / BPJS+Rujukan / Umum
   → Reminder bawa dokumen per tipe muncul
3. Verifikasi BPJS eligibilitas otomatis
4. Health anchor: alergi + kondisi + obat rutin
   (semua centang — zero ketik)
5. Nama panggilan + nama lengkap + tel. darurat
6. QR code REG-XXXX untuk ke loket

Rules:
- nickname → semua komunikasi
- full_name → resep & dokumen resmi
- Nama panggilan bisa diubah kapanpun
```

### Screen 2: Beranda
```
Konten:
- Greeting dengan nickname
- Antrian aktif hari ini (jika ada)
- Reminder obat hari ini
- Konten health companion (tips/mitos/insight)
  — tidak setiap hari, tidak predictable
- Konsultasi berikutnya (jika ada)
```

### Screen 3: Booking Dokter
```
Flow:
1. Filter: Semua / Hari ini / per Spesialisasi
2. List dokter dengan:
   - Nama + spesialisasi
   - Jadwal hari ini vs mendatang
   - Estimasi tunggu + jumlah antrian
   - Centang untuk pilih
3. Pilih hari (scroll horizontal)
   - Status: Buka / Penuh / Libur
4. Pilih jam slot
   - Visual: tersedia (sisa X) / penuh / selesai
5. Konfirmasi: dokter + hari + jam + tipe bayar
6. Tiket digital + nomor antrian

BARU — Reschedule & Cancel:
- Tombol reschedule di tiket aktif
- Pilih slot baru dari kalender
- Cancel dengan policy:
  BPJS: free cancel kapanpun
  Umum: H-1 free, H-0 ada biaya admin
- Konfirmasi cancel dengan warning
```

### Screen 4: Antrian Real-time
```
Tampil:
- Posisi pasien (angka besar)
- Total antrian hari ini
- Siapa yang sedang dilayani
- Estimasi waktu dipanggil
- Visual track antrian (done/active/waiting)
- Notif push saat "Anda selanjutnya"
- Notif saat dokter memanggil
```

### Screen 5: Rekam Medis & Riwayat
```
Tampil:
- Health anchor selalu di atas (alergi + kondisi)
- List kunjungan (terbaru di atas)
- Per kunjungan:
  → Dokter + tanggal + diagnosis
  → Resep digital dengan status
  → Vital signs saat kunjungan
  → Hasil lab & penunjang (BARU)
- Export PDF rekam medis (minor, sprint 2)
- Share ke dokter lain via QR (minor, sprint 2)
```

### Screen 6: Hasil Lab & Penunjang (BARU)
```
Tampil:
- List semua order lab per kunjungan
- Status: menunggu / proses / selesai
- Saat selesai:
  → Nilai hasil per parameter
  → Rentang normal dengan warna:
    Hijau = normal
    Kuning = perhatian
    Merah = di luar normal
  → Penjelasan dalam bahasa awam
  → Catatan dokter (jika ada)
  → Download PDF hasil asli
- Notif push saat hasil tersedia
- Pasien TIDAK bisa interpret sendiri
  tanpa catatan dokter
```

### Screen 7: Health Companion
```
Medication reminder:
- Jadwal dari resep aktif
- Tap ✓ yang satisfying
- Streak tracking
- Data kepatuhan → visible dokter

Content mix (tidak predictable):
Tips praktis:     ~40% dari hari aktif
Mitos buster:     ~15% (jeda min 5 hari)
Health insight:   ~8% (2x sebulan)
Targeted ads:     ~5% (max 2x/bulan)
Silence:          ~32% — reminder saja

Rules:
- Tidak ada konten Minggu
- Ads tidak pernah berurutan
- Insight bisa muncul kapan saja
- Konten dipilih berdasarkan:
  kondisi + obat aktif + waktu hari

Smartwatch:
- Sync 06.00 pagi — 1x sehari
- Morning baseline window
- iOS: Apple HealthKit
- Android: Google Health Connect
- Data: HR, SpO2, HRV, steps, sleep, BP
```

---

## 6. SUSTER APP — SCREENS MVP

### Screen 1: Dashboard Antrian Hari Ini
```
Tampil real-time:
- List semua pasien per dokter
- Status: menunggu / dipanggil / selesai
- Filter per dokter / poli
- Tombol panggil pasien berikutnya
```

### Screen 2: Registrasi Pasien
```
Flow:
1. Scan QR kode pasien (dari self-upload)
   ATAU input manual NIK
2. Data muncul otomatis dari OCR
3. Suster verifikasi sambil:
   - Pasien timbang badan
   - Suster input BB, TB, tensi, nadi, suhu
   - Suster tanya keluhan utama (3 field):
     → Keluhan (free text singkat)
     → Durasi (angka + satuan)
     → Tipe kunjungan (baru/kontrol/rujukan)
4. Konfirmasi → nomor antrian tercetak/digital
Total: 2-3 menit per pasien

BPJS Filter:
- Otomatis cek coverage saat registrasi
- Alert: "Tindakan X tidak ditanggung BPJS"
- Pasien harus acknowledge sebelum lanjut
```

### Screen 3: Input Vital Signs
```
Form:
BB (kg), TB (cm) → BMI auto
Tensi (mmHg), Nadi (/mnt)
Suhu (°C), SpO2 (%)
GDS (mg/dL) — opsional
```

### Screen 4: Kelola Jadwal Dokter
```
- Kalender jadwal per dokter
- Toggle buka/tutup per slot
- Block cuti / tidak hadir
- Set kapasitas per sesi
```

---

## 7. DOKTER APP — SCREENS MVP

### Screen 1: Dashboard Jadwal Hari Ini
```
Tampil:
- List pasien hari ini
- Status vital signs (sudah/belum diinput suster)
- Centang brief per pasien
- Tombol panggil berikutnya
```

### Screen 2: Rekam Medis Pasien
```
Selalu visible di atas (health anchor):
- 🔴 Alergi aktif
- 📋 Kondisi kronis
- 💊 Obat rutin

Alert otomatis:
- Saat input resep → cek alergi
- Riwayat relevan dengan keluhan hari ini
- Obat yang sedang diminum (interaksi)

Input via voice:
- Anamnesis → Web Speech API
- Diagnosis + terapi → voice
- Resep → voice (nama obat autocomplete)

Resep:
- Nama obat + BPJS formularium check
- Coverage alert per obat
```

### Screen 3: Order Lab & Penunjang
```
Dokter order:
- Pilih tipe: Lab / Radiologi / EKG / Lain
- Nama pemeriksaan
- Catatan klinis
- BPJS coverage check otomatis

Status update real-time ke pasien.
Input hasil oleh petugas lab.
Dokter tambah catatan interpretasi.
```

---

## 8. BPJS INTEGRATION

### Coverage Filter — Game Changer
```
Saat registrasi pasien BPJS:
Keluhan diinput → sistem query coverage rules
→ Tampil list:
  ✓ Konsultasi dokter spesialis: ditanggung
  ✓ Lab dasar: ditanggung
  ✗ MRI: tidak ditanggung (butuh indikasi)
  ⚠ Fisioterapi: max 6x per episode

Pasien acknowledge sebelum lanjut.
RS tidak perlu jelaskan manual.
Zero surprise di kasir.
```

### Alergi Alert saat Resep
```
Dokter input nama obat:
→ Sistem cek patient_allergies
→ Cek drug_class (golongan)
→ Jika match: alert merah muncul
→ Dokter bisa override dengan
  alasan (tercatat di audit log)
```

---

## 9. RESCHEDULE & CANCEL (Parameter Baru)

```
RESCHEDULE:
Pasien tap "Ubah Jadwal" di tiket
→ Pilih slot baru tersedia
→ Slot lama otomatis freed
→ Notif ke dokter & suster
→ Tiket baru digenerate

CANCEL:
Pasien tap "Batalkan"
→ Konfirmasi dengan warning
→ Policy applied otomatis:
  BPJS H-berapa pun: free
  Umum H-1: free
  Umum H-0: biaya admin (RS set)
→ Slot freed real-time
→ Notif ke semua pihak

SUSTER/ADMIN bisa cancel atas nama pasien
+ catat alasan
```

---

## 10. HASIL LAB & PENUNJANG (Parameter Baru)

```
FLOW:
Dokter order lab di app
→ Petugas lab terima order
→ Proses pemeriksaan
→ Petugas lab input hasil
→ Dokter review + tambah catatan
→ Pasien dapat notif: "Hasil tersedia"
→ Pasien buka app → lihat hasil

TAMPILAN PASIEN:
Per parameter:
- Nama pemeriksaan
- Nilai hasil
- Satuan
- Rentang normal
- Status visual: 🟢 Normal / 🟡 Perhatian / 🔴 Di luar normal
- Penjelasan awam (bukan diagnosa)
- Catatan dokter

ATURAN PENTING:
Hasil lab TIDAK bisa diinterpret
sebagai diagnosis oleh sistem.
Hanya dokter yang bisa
tambah catatan interpretasi.
Pasien selalu diarahkan untuk
diskusi dengan dokter.
```

---

## 11. NOTIFICATION SYSTEM — 20 Types

**Untuk Pasien:**
1. REGISTRATION_DONE — QR code siap
2. BOOKING_CONFIRMED — tiket + nomor antrian
3. RESCHEDULE_CONFIRMED — jadwal baru
4. CANCEL_CONFIRMED — pembatalan + policy
5. QUEUE_UPDATE — posisi antrian berubah
6. NEXT_UP — giliran berikutnya
7. DOCTOR_CALLING — dokter memanggil
8. CONSULTATION_DONE — rekam medis siap
9. PRESCRIPTION_READY — resep digital
10. LAB_ORDERED — pemeriksaan dipesan dokter
11. LAB_RESULT_READY — hasil tersedia
12. MEDICATION_REMINDER — waktu minum obat
13. FOLLOW_UP_REMINDER — H-3 kontrol berikutnya
14. BPJS_COVERAGE_ALERT — tindakan tidak covered

**Untuk Dokter:**
15. NEW_PATIENT — pasien baru antri
16. VITAL_READY — suster sudah input vital signs
17. LAB_RESULT_DONE — hasil lab masuk

**Untuk Suster:**
18. NEW_BOOKING — booking baru masuk
19. PATIENT_ARRIVED — pasien sudah scan QR

**Untuk Admin:**
20. DAILY_SUMMARY — laporan harian otomatis

---

## 12. CONTENT ENGINE — Health Companion

```
MIX per 30 hari (pasien kronis):
22x Reminder murni      (73%)
6x  Tips praktis        (20%)
3x  Mitos buster        (10%)
2x  Health insight      (7%)
2x  Targeted ads        (7%)

RULES:
- Tidak ada konten Minggu
- Tipe sama tidak boleh 2 hari berturut
- Ads jeda minimum 14 hari
- Mitos buster jeda minimum 5 hari
- Konten dipilih dari pool berdasarkan:
  condition_tags + drug_tags + time_of_day
- Silence adalah strategi — tidak forced

TARGETED ADS:
- Kurasi ketat: alat kesehatan, suplemen
  evidence-based, asuransi relevan
- Tidak pernah: obat-obatan, MLM,
  klaim penyembuhan
- Pasien bisa opt-out — tips tetap jalan
- Revenue: CPM Rp50-150k +
  revenue share dari pembelian
```

---

## 13. SPRINT PLAN — DEMO (4 Minggu)

### Minggu 1 — Foundation
- Setup Next.js + Supabase + Auth
- Role-based routing
- Database schema + dummy data
- Layout dasar semua role

### Minggu 2 — Patient Flow
- Onboarding + OCR mock
- BPJS verification mock
- Health anchor form (centang)
- Booking + antrian real-time
- Reschedule & cancel ✅ NEW

### Minggu 3 — Clinical Flow
- Vital signs input (suster)
- Rekam medis + voice input (dokter)
- Resep + BPJS check + alergi alert
- Lab order + hasil lab ✅ NEW
- Medication reminder + content engine

### Minggu 4 — Polish & Demo
- BPJS coverage filter di registrasi
- QR code flow
- Web push notifications
- Dummy data yang realistic
- Demo script 20 menit
- Deploy Vercel — semua fitur aktif

---

## 14. INFRASTRUCTURE

```
Target demo:  100 dummy pasien
Target live:  500-5000 pasien per RS

DB size:      ~540MB/tahun per 500 pasien
Cost/bulan:   Rp230.000-540.000

Services:
PostgreSQL (Supabase):  $25/bulan
Backend (Vercel):       $0 (hobby)
Storage (Cloudflare R2): $0 (<10GB)
Redis (Upstash):        $0 (free tier)
Total:                  ~$25/bulan
```

---

## 15. PORTFOLIO

Live demo URL (setelah deploy):
- `medicore.vercel.app` atau subdomain RS
- Demo: `demo.medicore.app`

Showcase di Flowo Studio:
- `flowo.vercel.app/medicore`

---

*MediCore PRD v1.0 — Disusun 1 Juni 2026*
*Siap untuk Demo Sprint — 4 minggu*
*Setelah kontrak: full scale development*
