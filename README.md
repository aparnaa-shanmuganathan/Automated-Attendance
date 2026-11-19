# Automated Face Recognition Attendance System

A real-time attendance system that recognises students from a webcam and records
subject-wise attendance, replacing manual roll calls. Faces are detected with a
Haar cascade and identified with an LBPH (Local Binary Patterns Histograms)
recogniser in OpenCV; attendance is written to per-subject CSV files with
per-student attendance percentages.

---

## Team and contributions

This was a two-person project.

| Area | Owner |
|------|-------|
| ML pipeline: face detection (Haar cascade), capture-time image quality checks, LBPH model training, real-time recognition with confidence thresholding, attendance recording to CSV and per-student percentage calculation | **Aparnaa A S** |
| Frontend / user interface for registering students and viewing attendance | **C Vishwak Sena** |

---

## How it works

```
Register  →  Capture 50 face samples  →  Train LBPH model  →  Take attendance  →  View / export
(ID, name)   Haar detection + quality      Trainner.yml        live recognition     % per student,
             checks, grayscale crops                            → subject CSV        Excel / PDF
```

1. **Register**: enter enrollment number and name. Duplicate enrollment numbers are rejected.
2. **Capture**: the webcam captures 50 grayscale face crops. Each frame must contain
   exactly one face and pass quality checks before it is saved:
   - face at least 100×100 px
   - mean brightness between 50 and 200
   - sharpness (variance of the Laplacian) at least 100, to reject blurred frames
3. **Train**: all face crops are labelled with the enrollment number and used to train an
   LBPH recogniser, saved to `TrainingImageLabel/Trainner.yml`.
4. **Take attendance**: for a subject, the camera runs for a fixed window
   (`ATTENDANCE_DURATION`, 20 s by default). Each detected face is passed to the
   recogniser; a match is accepted only when the LBPH distance is below
   `CONFIDENCE_THRESHOLD` (lower distance means a closer match). Each student is
   marked once per session, and the session is saved as
   `Attendance/<subject>/<subject>_<date>_<time>.csv`, with a backup copy.
5. **View attendance**: all sessions for a subject are merged, and each student's
   attendance percentage is calculated. The summary can be exported to Excel or PDF.

---

## Quick start

**Prerequisites:** Python 3.9+, a webcam.

```bash
git clone https://github.com/aparnaa-shanmuganathan/Automated-Attendance.git
cd Automated-Attendance
pip install -r requirements.txt
python attendance.py
```

The app creates `TrainingImage/`, `TrainingImageLabel/`, `StudentDetails/` and
`Attendance/` on first run. These hold personal data (face images, names, attendance)
and are excluded from git.

---

## Configuration (`config.py`)

| Setting | Default | Meaning |
|---------|---------|---------|
| `IMAGES_PER_STUDENT` | 50 | Face samples captured at registration |
| `CONFIDENCE_THRESHOLD` | 50 | Maximum LBPH distance accepted as a match (lower = stricter) |
| `SCALE_FACTOR` / `MIN_NEIGHBORS` | 1.2 / 5 | Haar cascade detection parameters |
| `ATTENDANCE_DURATION` | 20 s | Length of an attendance capture session |
| `CAMERA_INDEX` | 0 | Which camera to use |

---

## Project structure

```
attendance.py            # Main application entry point
takeImage.py             # Registration: face capture with quality checks
trainImage.py            # LBPH model training
automaticAttedance.py    # Real-time recognition and attendance recording
show_attendance.py       # Attendance summary and percentages
export_utils.py          # Excel / PDF export
config.py                # Tunable settings
logger_config.py         # Logging
haarcascade_frontalface_default.xml
backend/, frontend/      # Experimental web version (Flask + MongoDB API, Next.js UI)
```

---

## Tech stack

Python · OpenCV (Haar cascade, LBPH from `opencv-contrib-python`) · NumPy · pandas ·
Tkinter · Pillow · pyttsx3 (voice prompts) · openpyxl / reportlab (exports)

---

## Known limitations

- **No liveness detection.** A printed photo or phone screen of a registered student may
  be recognised. Anti-spoofing (blink or motion checks, depth camera) would be needed
  before relying on it to stop proxy attendance.
- LBPH is sensitive to large changes in lighting, pose and appearance; retrain after
  significant changes.
- Enrollment numbers must be numeric, because LBPH labels are integers.
- Face images are stored unencrypted on the local machine.

---

## Credits

- C Vishwak Sena ([@vishwaksen21](https://github.com/vishwaksen21)): frontend / user interface
