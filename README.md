# Penerapan BPM Lifecycle pada Proses Pendaftaran Schoters by Ruangguru

**Mata Kuliah:** Manajemen Proses Bisnis (MPB)  
**Kelompok:** 3  
**Program Studi:** S1 Sistem Informasi  
**Studi Kasus:** Schoters by Ruangguru — platform konsultasi & pendaftaran studi ke luar negeri

---

## Anggota Kelompok

| Nama | NIM |
|------|-----|
| Ghita Cahya Ramadhanti | 2310512078 |
| Kayla Alisha Rania | 2310512107 |
| Naufal Fahmi Prasetio | 2310512087 |
| Reyza Fajar Rabbani | 2310512122 |

---

## Deskripsi Proyek

Proyek ini menerapkan **BPM Lifecycle** (Discovery → Analysis → Design → Implementation → Monitoring → Improvement) pada **proses pendaftaran calon siswa (enrollment) Schoters by Ruangguru** — mulai dari pengajuan pendaftaran, verifikasi & kualifikasi lead, konsultasi, penilaian kelayakan otomatis (DMN), penawaran program, tanda tangan digital, pembayaran, hingga onboarding.

Fokus perbaikan dari proses **As-Is** ke **To-Be** adalah otomatisasi keputusan kelayakan menggunakan **DMN**, penambahan loop revisi dokumen, serta orkestrasi lintas peran (Student, Consultant, System, Finance) dalam satu proses yang dapat dieksekusi.

**Tools yang digunakan:**
- **Camunda Modeler** — pembuatan model BPMN, DMN, dan Form
- **CIB seven BPMS 2.2.0** (engine Camunda Platform 7) — deployment, Tasklist, dan Cockpit
- **REST Engine API** (`http://localhost:8080/engine-rest`) — monitoring & query instance

---

## Struktur Repository

```
kelompok_3_SchotersByRuangguru/
├── laporan/
│   └── laporan-proyek-bpm.pdf
├── model/
│   ├── schoters_as_is_final.bpmn      # Model proses As-Is (dokumentasi)
│   ├── schoters_to_be_final.bpmn      # Model proses To-Be (executable)
│   ├── schoters_decision.dmn          # Decision table kelayakan
│   └── forms/
│       ├── form_pendaftaran.form
│       ├── form_verifikasi.form
│       ├── form_review_penawaran.form
│       └── form_tanda_tangan.form
├── screenshots/
│   ├── deployment/                    # Bukti deploy BPMN/DMN/Form
│   ├── tasklist/                      # Form tiap user task
│   ├── cockpit/                       # Dashboard, process definitions,
│   │   └── process-instance-history/  #   & riwayat instance per skenario
│   └── simulation/                    # Bukti tiap skenario dijalankan
│       ├── Skenario 1 - Approved/
│       ├── Skenario 2 - Revision/
│       └── Skenario 3 - Rejected/
├── video/
│   └── link-video.txt
└── README.md
```

---

## Model Proses

| File | Keterangan |
|------|------------|
| `model/schoters_as_is_final.bpmn` | Proses pendaftaran kondisi awal (As-Is), bersifat dokumentatif (`isExecutable=false`). |
| `model/schoters_to_be_final.bpmn` | Proses usulan (To-Be) yang dapat dieksekusi. Key proses: **`Process_Schoters_TB`**. Terdiri atas 4 lane: **Student, Consultant, System, Finance**, dengan loop revisi dokumen dan sub-proses onboarding. |
| `model/schoters_decision.dmn` | Decision table penilaian kelayakan. Key decision: **`Decision_Kelayakan`**. |

**Alur ringkas To-Be:** Submit Enrollment Application → Verify & Qualify Lead → Conduct Online Consultation → Upload Supporting Documents → **Evaluate Eligibility (DMN)** → *gateway* keputusan:
- **Approved** → Prepare Program Recommendation → Review Digital Offer Letter → Sign Digital Agreement → Complete Online Payment → Confirm Payment Settlement → Schedule Onboarding Video Call → **Student Successfully Enrolled**
- **Revision** → Revise & Resubmit Application Data → kembali ke Upload Supporting Documents (loop)
- **Rejected** → **Application Rejected**

---

## Form (User Tasks)

| File | User Task | Field Utama |
|------|-----------|-------------|
| `form_pendaftaran.form` | Submit Enrollment Application | `fullName`, `email`, `programLevel` (Foundation/Bachelor/Master), `targetCountry`, `programValue` |
| `form_verifikasi.form` | Verify & Qualify Lead | `documentsComplete` (checkbox), `riskLevel` (Low/Medium/High) |
| `form_review_penawaran.form` | Review Digital Offer Letter | `offerAccepted` (checkbox) |
| `form_tanda_tangan.form` | Sign Digital Agreement (e-Signature) | `signatureConfirmed` (checkbox), `signDate` (date) |

---

## Variabel Proses Penting

| Variabel | Tipe | Sumber | Keterangan |
|----------|------|--------|------------|
| `fullName` | String | Form Pendaftaran | Nama calon siswa |
| `email` | String | Form Pendaftaran | Email pendaftar |
| `programLevel` | String | Form Pendaftaran | Foundation / Bachelor / Master |
| `targetCountry` | String | Form Pendaftaran | Negara tujuan studi |
| `programValue` | Number | Form Pendaftaran | Nilai program (input DMN) |
| `documentsComplete` | Boolean | Form Verifikasi | Kelengkapan dokumen (input DMN) |
| `riskLevel` | String | Form Verifikasi | Low / Medium / High (input DMN) |
| `decision` | String | DMN | Output keputusan: Approved / Revision / Rejected |
| `offerAccepted` | Boolean | Form Review Penawaran | Persetujuan penawaran |
| `signatureConfirmed` | Boolean | Form Tanda Tangan | Konfirmasi tanda tangan digital |
| `signDate` | Date | Form Tanda Tangan | Tanggal tanda tangan |

---

## DMN Decision Table

**File:** `model/schoters_decision.dmn`  
**Decision ID:** `Decision_Kelayakan`  
**Tabel:** `DT_Kelayakan` — **Hit Policy: UNIQUE**

| documentsComplete | riskLevel | programValue | decision |
|:---:|:---:|:---:|:---:|
| false | — | — | **Revision** |
| true | Low | — | **Approved** |
| true | High | — | **Rejected** |
| true | Medium | `<= 30.000.000` | **Approved** |
| true | Medium | `> 30.000.000` | **Rejected** |

> Logika: dokumen tidak lengkap selalu minta revisi; risiko rendah disetujui; risiko tinggi ditolak; risiko sedang ditentukan oleh nilai program (ambang Rp30.000.000).

---

## Hasil Simulasi

Ketiga skenario inti dijalankan penuh di CIB seven dan dibuktikan lewat Tasklist, Cockpit, dan Process Instance History (lihat folder `screenshots/`).

| Skenario | Input (documentsComplete / riskLevel / programValue) | Output DMN | Akhir Proses |
|----------|------------------------------------------------------|:---:|--------------|
| **Skenario 1 — Approved** | true / Low / 50.000.000 | Approved | Student Successfully Enrolled |
| **Skenario 2 — Revision** | false (→ diperbaiki → true) / Low | Revision → Approved | Student Successfully Enrolled (setelah loop revisi) |
| **Skenario 3 — Rejected** | true / High | Rejected | Application Rejected |

**Uji batas tambahan (risiko Medium):**

| Input | Output DMN | Keterangan |
|-------|:---:|------------|
| true / Medium / 25.000.000 | Approved | Nilai ≤ ambang Rp30jt |
| true / Medium / 50.000.000 | Rejected | Nilai > ambang Rp30jt |

---

## Cara Menjalankan

### 1. Deploy ke CIB seven
Buka **Camunda/CIB seven Modeler** → **Deploy**, lalu unggah seluruh isi folder `model/` sekaligus (BPMN To-Be, DMN, dan semua `.form`) agar referensi DMN dan form ikut ter-deploy.

### 2. Start Process Instance
Lewat **Tasklist** (`Start Process` → *Schoters Enrollment Process (To-Be)*) lalu isi Form Pendaftaran, atau lewat REST API:
```bash
curl -X POST http://localhost:8080/engine-rest/process-definition/key/Process_Schoters_TB/start \
  -H "Content-Type: application/json" \
  -d '{}'
```

### 3. Complete Task
Selesaikan tiap user task melalui **Tasklist** sesuai perannya (Student / Consultant / Finance).

### 4. Monitor via Cockpit
Buka **Cockpit** → *Processes* untuk melihat process definitions, dan buka tiap instance untuk melihat jalur eksekusi, **Variables**, serta **Called Decision Instances** (hasil evaluasi DMN).

---

## BPM Lifecycle

| Fase | Output |
|------|--------|
| Discovery | Model As-Is (`schoters_as_is_final.bpmn`) |
| Analysis | Identifikasi bottleneck & keputusan manual pada proses As-Is |
| Design | Model To-Be (`schoters_to_be_final.bpmn`) + DMN + Form |
| Implementation | Deployment ke CIB seven (lihat `screenshots/deployment/`) |
| Monitoring | Simulasi 3 skenario + Cockpit history (`screenshots/simulation/`, `screenshots/cockpit/`) |
| Improvement | Otomatisasi keputusan via DMN & loop revisi dokumen |
