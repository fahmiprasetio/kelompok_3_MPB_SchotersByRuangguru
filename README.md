# Penerapan BPM Lifecycle pada Proses Bisnis Schoters by Ruangguru

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

Proyek ini menerapkan **BPM Lifecycle** (Discovery → Analysis → Design → Implementation → Monitoring → Improvement) pada **tiga proses bisnis** Schoters by Ruangguru. Untuk setiap proses dibuat model **As-Is** (dokumentatif) dan **To-Be** (executable), dilengkapi **DMN** untuk otomatisasi keputusan dan **Form** untuk tiap user task.

Ketiga proses tersebut:

| No | Proses | Folder | Fokus |
|----|--------|--------|-------|
| 1 | **Pendaftaran Calon Siswa (Enrollment)** | `model/01_pendaftaran/` | Pengajuan pendaftaran → kualifikasi lead → konsultasi → penilaian kelayakan (DMN) → penawaran → tanda tangan digital → pembayaran → onboarding |
| 2 | **Aplikasi Universitas (University Application)** | `model/02_aplikasi_universitas/` | Permintaan aplikasi → penyiapan & review dokumen/esai → penilaian kesiapan (DMN) → pengiriman aplikasi ke universitas |
| 3 | **Refund / Pembatalan Program** | `model/03_refund_pembatalan/` | Pengajuan refund → tinjauan → evaluasi kelayakan refund (DMN) → proses refund penuh/sebagian/tolak → notifikasi |

Fokus perbaikan dari **As-Is** ke **To-Be** pada setiap proses: otomatisasi keputusan menggunakan **DMN**, penambahan loop revisi, serta orkestrasi lintas peran dalam satu proses yang dapat dieksekusi.

**Tools yang digunakan:**
- **Camunda Modeler** — pembuatan model BPMN, DMN, dan Form
- **CIB seven BPMS** (engine Camunda Platform 7) — deployment, Tasklist, dan Cockpit
- **REST Engine API** (`http://localhost:8080/engine-rest`) — monitoring & query instance

---

## Struktur Repository

```
kelompok_3_MPB_SchotersByRuangguru/
├── laporan/
│   └── laporan-proyek-bpm.pdf
├── model/
│   ├── 01_pendaftaran/
│   │   ├── schoters_as_is_final.bpmn        # As-Is (dokumentatif)
│   │   ├── schoters_to_be_final.bpmn        # To-Be (executable)
│   │   ├── schoters_decision.dmn            # DMN kelayakan pendaftaran
│   │   ├── form_pendaftaran.form
│   │   ├── form_verifikasi.form
│   │   ├── form_review_penawaran.form
│   │   └── form_tanda_tangan.form
│   ├── 02_aplikasi_universitas/
│   │   ├── aplikasi_universitas_as_is.bpmn  # As-Is (dokumentatif)
│   │   ├── aplikasi_universitas_to_be.bpmn  # To-Be (executable)
│   │   ├── aplikasi_universitas_decision.dmn# DMN kesiapan aplikasi
│   │   ├── form_app_request.form
│   │   ├── form_review_docs.form
│   │   └── form_admission.form
│   └── 03_refund_pembatalan/
│       ├── refund_pembatalan_as_is.bpmn     # As-Is (dokumentatif)
│       ├── refund_pembatalan_to_be.bpmn     # To-Be (executable)
│       ├── refund_pembatalan_decision.dmn   # DMN kelayakan refund
│       ├── form_refund_request.form
│       └── form_refund_review.form
├── screenshots/
│   ├── 01_pendaftaran/
│   ├── 02_aplikasi_universitas/
│   └── 03_refund_pembatalan/
│       ├── deployment/   # Bukti deploy BPMN/DMN/Form
│       ├── tasklist/     # Form tiap user task
│       ├── cockpit/      # Dashboard, process definitions, riwayat instance
│       └── simulation/   # Bukti tiap skenario dijalankan
├── video/
│   └── link-video.txt
└── README.md
```

---

## Proses 1 — Pendaftaran Calon Siswa

| Item | Keterangan |
|------|------------|
| As-Is | `model/01_pendaftaran/schoters_as_is_final.bpmn` (`isExecutable=false`) |
| To-Be | `model/01_pendaftaran/schoters_to_be_final.bpmn` — key proses **`Process_Schoters_TB`**, lane: Student, Consultant, System, Finance |
| DMN | `model/01_pendaftaran/schoters_decision.dmn` — key **`Decision_Kelayakan`** |

**Alur ringkas To-Be:** Submit Enrollment Application → Verify & Qualify Lead → Conduct Online Consultation → Upload Supporting Documents → **Evaluate Eligibility (DMN)** → keputusan:
- **Approved** → Prepare Program Recommendation → Review Digital Offer Letter → Sign Digital Agreement → Complete Online Payment → Confirm Payment Settlement → Schedule Onboarding → **Student Successfully Enrolled**
- **Revision** → Revise & Resubmit Application Data → kembali ke Upload Supporting Documents (loop)
- **Rejected** → **Application Rejected**

**DMN `Decision_Kelayakan` (Hit Policy: UNIQUE)** — input `documentsComplete`, `riskLevel`, `programValue`; output `decision`:

| documentsComplete | riskLevel | programValue | decision |
|:---:|:---:|:---:|:---:|
| false | — | — | **Revision** |
| true | Low | — | **Approved** |
| true | High | — | **Rejected** |
| true | Medium | `<= 30.000.000` | **Approved** |
| true | Medium | `> 30.000.000` | **Rejected** |

**Form:** `form_pendaftaran` (Submit Enrollment Application), `form_verifikasi` (Verify & Qualify Lead), `form_review_penawaran` (Review Digital Offer Letter), `form_tanda_tangan` (Sign Digital Agreement).

---

## Proses 2 — Aplikasi Universitas

| Item | Keterangan |
|------|------------|
| As-Is | `model/02_aplikasi_universitas/aplikasi_universitas_as_is.bpmn` — key **`Process_AppUniv_AS`** (`isExecutable=false`) |
| To-Be | `model/02_aplikasi_universitas/aplikasi_universitas_to_be.bpmn` — key **`Process_AppUniv_TB`** |
| DMN | `model/02_aplikasi_universitas/aplikasi_universitas_decision.dmn` — key **`Decision_Aplikasi`** |

**Fokus proses:** mahasiswa mengajukan permintaan aplikasi universitas, konsultan/tim menyiapkan & mereview dokumen dan esai, lalu sistem menilai kesiapan aplikasi via DMN sebelum aplikasi dikirim ke universitas tujuan. Terdapat loop revisi dokumen bila belum siap.

**DMN `Decision_Aplikasi` (Hit Policy: UNIQUE)** — input `essayReady`, `gpa`, `targetTier`; output `appDecision`:

| essayReady | gpa | targetTier | appDecision |
|:---:|:---:|:---:|:---:|
| false | — | — | **Revise Documents** |
| true | `>= 3.5` | — | **Submit** |
| true | `< 3.5` | Top | **Adjust Target** |
| true | `< 3.5` | Mid, Safe | **Submit** |

**Form:** `form_app_request` (permintaan aplikasi), `form_review_docs` (review dokumen & esai), `form_admission` (pengiriman aplikasi/admission).

---

## Proses 3 — Refund / Pembatalan Program

| Item | Keterangan |
|------|------------|
| As-Is | `model/03_refund_pembatalan/refund_pembatalan_as_is.bpmn` — key **`Process_Refund_AS`** (`isExecutable=false`), 4 lane: Mahasiswa, Customer Service, Finance, Manager |
| To-Be | `model/03_refund_pembatalan/refund_pembatalan_to_be.bpmn` — key **`Process_Refund_TB`**, 3 lane: Mahasiswa, Tim Finance, Sistem Schoters |
| DMN | `model/03_refund_pembatalan/refund_pembatalan_decision.dmn` — key **`Decision_Refund`** |

**Alur ringkas To-Be:** Ajukan Permintaan Refund → Validasi Otomatis → *gateway* Data Lengkap? (loop perbaikan data bila tidak lengkap) → Tinjau Permintaan Refund → **Evaluasi Kelayakan Refund (DMN)** → keputusan:
- **Full Refund** → Proses Refund Penuh → Catat Transaksi → Kirim Notifikasi Refund Selesai → **Refund Selesai**
- **Partial Refund** → Setujui Refund Sebagian → Proses Refund Sebagian → Catat Transaksi → Notifikasi → **Refund Selesai**
- **No Refund** → Kirim Notifikasi Penolakan → **Refund Ditolak**

**DMN `Decision_Refund` (Hit Policy: UNIQUE)** — input `withinCoolingOff`, `reasonCategory`, `serviceUsedPercent`; output `refundDecision`:

| withinCoolingOff | reasonCategory | serviceUsedPercent | refundDecision |
|:---:|:---:|:---:|:---:|
| true | — | — | **Full Refund** |
| false | Valid | `<= 50` | **Partial Refund** |
| false | Valid | `> 50` | **No Refund** |
| false | Invalid | — | **No Refund** |

> Logika: dalam masa cooling-off selalu refund penuh; di luar cooling-off, alasan valid dengan pemakaian layanan ≤ 50% mendapat refund sebagian, > 50% atau alasan tidak valid tidak mendapat refund.

**Form:** `form_refund_request` (pengajuan refund), `form_refund_review` (tinjauan refund oleh Finance).

---

## Cara Menjalankan (per Proses)

### 1. Deploy ke CIB seven
Buka **Camunda/CIB seven Modeler** → **Deploy**, lalu unggah seluruh isi folder proses terkait sekaligus (BPMN To-Be + DMN + semua `.form`) agar referensi DMN dan form ikut ter-deploy.

### 2. Start Process Instance
Lewat **Tasklist** (`Start Process`) lalu isi form awal, atau lewat REST API. Contoh untuk proses pendaftaran:
```bash
curl -X POST http://localhost:8080/engine-rest/process-definition/key/Process_Schoters_TB/start \
  -H "Content-Type: application/json" \
  -d '{}'
```
Ganti key proses sesuai kebutuhan: `Process_Schoters_TB`, `Process_AppUniv_TB`, atau `Process_Refund_TB`.

### 3. Complete Task
Selesaikan tiap user task melalui **Tasklist** sesuai perannya.

### 4. Monitor via Cockpit
Buka **Cockpit** → *Processes* untuk melihat process definitions, dan buka tiap instance untuk melihat jalur eksekusi, **Variables**, serta **Called Decision Instances** (hasil evaluasi DMN).

---

## BPM Lifecycle

| Fase | Output |
|------|--------|
| Discovery | Model As-Is tiap proses (`*_as_is*.bpmn`) |
| Analysis | Identifikasi bottleneck & keputusan manual pada proses As-Is |
| Design | Model To-Be (`*_to_be*.bpmn`) + DMN + Form |
| Implementation | Deployment ke CIB seven (lihat `screenshots/<proses>/deployment/`) |
| Monitoring | Simulasi skenario + Cockpit history (`screenshots/<proses>/simulation/`, `cockpit/`) |
| Improvement | Otomatisasi keputusan via DMN & loop revisi pada tiap proses |
