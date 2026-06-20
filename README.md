# Penerapan BPM Lifecycle pada Proses Bisnis Schoters by Ruangguru

**Mata Kuliah:** Manajemen Proses Bisnis (MPB) · **Kelompok:** 3 · **Prodi:** S1 Sistem Informasi  
**Studi Kasus:** Schoters by Ruangguru — platform konsultasi & pendaftaran studi ke luar negeri

## Anggota Kelompok

| Nama | NIM |
|------|-----|
| Ghita Cahya Ramadhanti | 2310512078 |
| Kayla Alisha Rania | 2310512107 |
| Naufal Fahmi Prasetio | 2310512087 |
| Reyza Fajar Rabbani | 2310512122 |

## Deskripsi

Proyek ini menerapkan **BPM Lifecycle** pada **tiga proses bisnis** Schoters by Ruangguru. Tiap proses memiliki model **As-Is** (dokumentatif) dan **To-Be** (executable), dilengkapi **DMN** untuk otomatisasi keputusan dan **Form** untuk user task.

| No | Proses | Folder |
|----|--------|--------|
| 1 | Pendaftaran Calon Siswa | `model/01_pendaftaran/` |
| 2 | Aplikasi Universitas | `model/02_aplikasi_universitas/` |
| 3 | Refund / Pembatalan Program | `model/03_refund_pembatalan/` |

**Tools:** Camunda Modeler (BPMN, DMN, Form) dan CIB seven BPMS (Camunda Platform 7) untuk deployment, Tasklist, dan Cockpit.

## Struktur Repository

```
kelompok_3_MPB_SchotersByRuangguru/
├── model/                 # BPMN (as-is & to-be), DMN, dan form per proses
│   ├── 01_pendaftaran/
│   ├── 02_aplikasi_universitas/
│   └── 03_refund_pembatalan/
├── screenshots/           # Bukti deploy, tasklist, cockpit, simulasi per proses
├── laporan/               # Laporan proyek
├── video/                 # Link video presentasi
└── README.md
```

## Key Proses (To-Be)

| Proses | Key BPMN | Key DMN |
|--------|----------|---------|
| Pendaftaran | `Process_Schoters_TB` | `Decision_Kelayakan` |
| Aplikasi Universitas | `Process_AppUniv_TB` | `Decision_Aplikasi` |
| Refund / Pembatalan | `Process_Refund_TB` | `Decision_Refund` |

## Cara Menjalankan

1. **Deploy** — di Camunda/CIB seven Modeler, unggah seluruh isi folder proses terkait sekaligus (BPMN To-Be + DMN + form).
2. **Start** — jalankan instance lewat Tasklist (`Start Process`) atau REST API:
   ```bash
   curl -X POST http://localhost:8080/engine-rest/process-definition/key/<KEY_PROSES>/start \
     -H "Content-Type: application/json" -d '{}'
   ```
3. **Complete Task** — selesaikan user task melalui Tasklist.
4. **Monitor** — pantau jalur eksekusi, variables, dan hasil DMN melalui Cockpit.

## BPM Lifecycle

Discovery (As-Is) → Analysis → Design (To-Be + DMN + Form) → Implementation (deploy) → Monitoring (simulasi + Cockpit) → Improvement (otomatisasi keputusan via DMN & loop revisi). Bukti tiap fase ada di folder `screenshots/`.
