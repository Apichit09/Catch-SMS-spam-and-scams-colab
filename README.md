# README — Thai SMS Spam/Scam Analyzer (Colab + Flask + LIME)

โปรเจกต์สำหรับสร้างระบบจำแนกข้อความ SMS ภาษาไทยเป็น 4 กลุ่ม: `OK`, `OTP`, `spam`, `scam` พร้อมอธิบายเหตุผลการทำนายด้วย **LIME** และมีหน้าเว็บ **Flask** ให้ทดสอบแบบอินเตอร์แอคทีฟ

---

## ฟีเจอร์หลัก

* โหลดชุดข้อมูลจาก repo: `ssivakorn/Thai-SMS-spam-filter`
* สร้างฟีเจอร์ทั้งเชิงสถิติ/แพทเทิร์น/คีย์เวิร์ด + `processed_message` (ตัดอีโมจิ/สต็อปเวิร์ด/เว้นวรรค)
* ทดลองสัดส่วน Train/Test: **60:40**, **70:30**, **80:20**
* เปรียบเทียบ 4 โมเดล: Logistic Regression, Random Forest, MLP (Neural Network), Naive Bayes
* **ปรับจูนพารามิเตอร์ (GridSearchCV)** สำหรับ MLP
* สร้าง **Production pipeline** (TF-IDF + Numeric features + StandardScaler + **SMOTE** + Tuned MLP) และบันทึกโมเดล
* ฟังก์ชัน `analyze_and_log_sms(...)` สำหรับทำนาย + อธิบายเหตุผลด้วย LIME + เขียน log
* เว็บเดโม่ **Flask** + **Chart.js** + **AOS** พร้อมแสดงความน่าจะเป็น/เหตุผล/ฟีเจอร์
* รองรับเปิดสาธารณะด้วย **ngrok** (บน Colab)

---

## สภาพแวดล้อมที่รองรับ

* **Google Colab** (โค้ดนี้เขียนมาเพื่อ Colab เป็นหลัก)
* Python 3.9+ (ถ้ารันในเครื่อง คุณต้องปรับพาธ/การติดตั้งแพ็กเกจเล็กน้อยเอง — มีคำแนะนำด้านล่าง)

---

## การติดตั้ง (บน Colab)

มีคำสั่งติดตั้งแพ็กเกจแล้ว :

```python
!pip install pythainlp -q
!pip install lime -q
!pip install imbalanced-learn -q
```

ช่วงท้ายสำหรับเว็บ :

```python
!pip install flask pandas pyngrok pythainlp lime scikit-learn imbalanced-learn -q
```

> หมายเหตุ : หากเจอปัญหาเวอร์ชัน ให้รีสตาร์ท runtime แล้วรันใหม่

---

## ข้อมูลเข้า (Dataset)

* ระบบจะ `git clone` repo: `https://github.com/ssivakorn/Thai-SMS-spam-filter.git`
* อ่านไฟล์ `.txt` ภายใต้ `data/raw_data/` ทุกไฟล์ แล้วรวมเป็น DataFrame ที่มีคอลัมน์ `label`, `message`

---

## โฟลว์การทำงานใน Colab (สรุป)

1. **เชื่อม Google Drive** และกำหนดโฟลเดอร์หลัก

* โค้ดต้นฉบับทำงานโดย **mount Drive** แล้วตั้ง `DRIVE_PROJECT_PATH`
* ไฟล์ผลลัพธ์/โมเดล/เว็บ (template) จะถูกเก็บไว้ใน:

  ```
  /content/drive/MyDrive/SMS_Scam_Final_Project/
  ```

2. **โหลดข้อมูล + สร้าง DataFrame**
3. **สร้างฟีเจอร์** ด้วย `create_features(df)`
4. **Encoding label** และบันทึก `label_encoder.joblib` ลง Drive
5. **ทดลองสัดส่วน/โมเดล** แล้วสรุป Accuracy
6. **แยก Train/Test (70:30)** และสร้าง `TF-IDF`
7. **เทรนโมเดลทุกตัว + แสดง Classification Report**
8. **Hyperparameter Tuning (MLP)** ด้วย `GridSearchCV`
9. **สร้าง Production Pipeline** รวม:

   * `ColumnTransformer`: `TfidfVectorizer(tokenizer=word_tokenize)` สำหรับข้อความ + `StandardScaler()` สำหรับตัวเลข
   * `SMOTE` เพื่อแก้ class imbalance
   * `clf = best_tuned_model` (MLP ที่จูนแล้ว)
10. **บันทึก Pipeline** เป็น `final_sms_classifier_production.joblib` ลง Drive
11. **ฟังก์ชันทำนาย** `analyze_and_log_sms(...)`:

    * โหลด pipeline + label encoder
    * สร้าง features
    * ทำนาย & LIME explain
    * บันทึก log เป็น CSV
12. **ทดสอบบนชุดข้อความตัวอย่าง**
13. **ส่วนเว็บ (Flask)**:

    * สร้างไฟล์ `templates/index.html` (บน Drive)
    * รัน `app.py`, เปิดพอร์ต 5000
    * ใช้ `pyngrok` เปิดลิงก์สาธารณะ

---

## ไฟล์/อาร์ติแฟกต์ที่บันทึก

บน Drive (ค่าเริ่มต้น) :

* `label_encoder.joblib`
* `final_sms_classifier_production.joblib`
* `prediction_log.csv`
* `templates/index.html`
* `app.py`

> ถ้าอยาก **บันทึกลงเครื่อง** (ไม่ใช้ Drive): เปลี่ยนตัวแปร `DRIVE_PROJECT_PATH` ให้เป็นโฟลเดอร์โลคอล (เช่น `BASE_DIR = Path.cwd() / "SMS_Scam_Final_Project"`), และแก้ทุกจุดที่อ้างถึง `DRIVE_PROJECT_PATH` ให้ใช้ `BASE_DIR` แทน

---

## การรันเว็บเดโม่

1. ติดตั้งและตั้งค่า ngrok
2. สร้าง `templates/index.html`
3. รัน `app.py` และเชื่อม `ngrok`:

   * ระบบพิมพ์ลิงก์ public ออกมา (`Click here!: http(s)://...`)
4. เปิดเว็บ → วางข้อความ SMS → กด “วิเคราะห์ข้อความ”

---


## รายละเอียดฟีเจอร์ที่สกัด (ตัวอย่าง)

* `message_length`, `digit_count`, `uppercase_ratio`, `repeated_punctuation_count`
* `url_count`, `has_short_url`, `has_phone`
* คีย์เวิร์ดบ่งชี้: `has_line_keyword`, `advertisement_words_count`, `is_known_brand`,
  `is_gambling_keyword`, `is_loan_keyword`, `is_job_offer_keyword`, `emotional_words_count`
* `obfuscated_word_count` (คำผสมไทย-อังกฤษเพื่อเลี่ยงฟิลเตอร์)
* `processed_message` (หลังตัดอีโมจิ/สต็อปเวิร์ด) และ `word_diversity_ratio`

---

## การปรับจูนพารามิเตอร์ (Hyperparameter Tuning)

ใช้ `GridSearchCV` กับ **MLPClassifier**:

* `hidden_layer_sizes`: `(50,)`, `(100,)`, `(50, 25)`
* `activation`: `tanh`, `relu`
* `alpha`: `0.0001`, `0.05`
* `learning_rate`: `constant`, `adaptive`
* `cv=3`, `n_jobs=-1`

> หลังจูน จะได้ `best_tuned_model` แล้วนำไปใช้ใน Production pipeline

---

## ข้อควรทราบ

* **LIME** อาจช้าบนข้อความยาว/บางสภาพแวดล้อม (โดยเฉพาะ Colab ฟรี)
* ข้อมูลดิบภายนอกอาจเปลี่ยนได้—หาก repo ต้นทางเปลี่ยนโครงสร้างไฟล์ ให้ปรับพาธในโค้ดโหลดข้อมูล
* หากภาษาไทย tokenization ไม่แม่นยำในบางข้อความ ลองพิจารณา engine/tokenizer อื่นของ `pythainlp`

---

## การรันบนเครื่อง (ไม่ใช้ Drive/Colab)

1. สร้าง `venv` และติดตั้งแพ็กเกจ:

   ```bash
   pip install pythainlp lime imbalanced-learn scikit-learn pandas numpy flask pyngrok joblib
   ```
2. **แก้พาธ**:

   * แทนที่ `DRIVE_PROJECT_PATH` ด้วยโฟลเดอร์โลคอล (เช่น `BASE_DIR = Path.cwd()/ "SMS_Scam_Project"`)
   * เปลี่ยนทุกการอ้าง `DRIVE_PROJECT_PATH` → `BASE_DIR`
3. รันบล็อกเทรนในโน้ตบุ๊ก (หรือดัดแปลงเป็นสคริปต์ `.py`)
4. รันส่วน Flask (`app.py`) → เปิดที่ `http://127.0.0.1:5000`
   *(ถ้าต้องออกอินเทอร์เน็ต ใช้ ngrok และตั้ง `NGROK_AUTHTOKEN` เป็น environment variable)*

---

## ไลเซนส์/เครดิต

* ข้อมูลดิบ: [ssivakorn/Thai-SMS-spam-filter](https://github.com/ssivakorn/Thai-SMS-spam-filter)
* ไลบรารี: `pythainlp`, `scikit-learn`, `imbalanced-learn`, `lime`, `flask`, `pyngrok`, `pandas`, `numpy`

---
