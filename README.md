

# 🔥 Automated XSS Hunting Pipeline (Katana → Dalfox)

This repository documents a **real-world automated workflow** for discovering **reflected XSS candidates** using modern recon and filtering tools.

> ⚠️ Use **only on targets you are authorized to test**.


## 🧰 Tools Used

* **Katana** – Web crawler
* **uro** – URL normalizer & deduplicator
* **Gxss** – Parameter injection helper
* **kxss** – Reflection detector
* **Dalfox** – Advanced XSS scanner


## 📌 Step-by-Step Pipeline


### 1️⃣ Crawl live subdomains with Katana

```bash
katana -list monzolive.txt -jc -kf all -d 3 -silent > katana_monzo.txt
```

**What this does:**

* `-list monzolive.txt` → Input file containing live subdomains
* `-jc` → Extract URLs from JavaScript
* `-kf all` → Crawl all known file types
* `-d 3` → Crawl depth (3 levels deep)
* `-silent` → No banner, clean output

📁 **Output:** `katana_monzo.txt` (all discovered URLs)

### 2️⃣ Extract URLs with parameters

```bash
grep "?" katana_monzo.txt > urls_with_params.txt
```

**Why:**

* XSS requires **parameters**
* This filters only URLs containing `?`

📁 **Output:** `urls_with_params.txt`

---

### 3️⃣ Normalize & deduplicate URLs

```bash
cat urls_with_params.txt | uro > cleanparams.txt
```

**What uro does:**

* Removes duplicate parameters
* Normalizes URL structure
* Reduces massive noise

📁 **Output:** `cleanparams.txt`

---

### 4️⃣ Inject XSS marker payload (Gxss)

```bash
cat cleanparams.txt | Gxss > gxss.txt
```

**Why Gxss:**

* Replaces parameter values with `Gxss`
* Helps detect reflection points

📁 **Output:** `gxss.txt`

---

### 5️⃣ Detect reflected parameters (kxss)

```bash
cat gxss.txt | kxss > reflected.txt
```

**What kxss does:**

* Sends requests
* Detects **reflected parameters**
* Shows filtering behavior

📁 **Output:** `reflected.txt`

Example:

```
URL: https://target.com/search?q=Gxss
Param: q
Unfiltered: ['<', '>', '"']
```

---

### 6️⃣ Extract only the vulnerable URLs

```bash
cat reflected.txt | grep -oP 'URL: \K\S+' > urls_only.txt
```

**Why:**

* Removes extra metadata
* Keeps only testable URLs

📁 **Output:** `urls_only.txt`

---

### 7️⃣ Clean injected payload marker

```bash
sed 's/Gxss//g' urls_only.txt > urls_clean.txt
```

**Purpose:**

* Removes `Gxss` placeholder
* Prepares URLs for real payload testing

📁 **Output:** `urls_clean.txt`

---

### 8️⃣ (Optional) Noise reduction

```bash
cat urls_clean.txt \
| grep -v "community.monzo.com/t/" \
| grep -v "page=" \
| sort -u \
> final_xss_targets.txt
```

**Why optional filtering:**

* Removes forum threads
* Removes pagination parameters
* Keeps high-value targets only

📁 **Final Targets:** `final_xss_targets.txt`

---

## 🚀 Final XSS Scanning with Dalfox

```bash
dalfox file final_xss_targets.txt \
  --custom-payload /home/alhamr/Downloads/xss_payloads.txt \
  --skip-mining-dom \
  --skip-mining-dict \
  --worker 50 \
  -o dalfox_results.txt
```

**Dalfox flags explained:**

* `file` → Scan URLs from file
* `--custom-payload` → Use your own payload list
* `--skip-mining-dom` → Faster scan (no DOM mining)
* `--skip-mining-dict` → Skip wordlist fuzzing
* `--worker 50` → High concurrency
* `-o` → Save results

📁 **Output:** `dalfox_results.txt`

```
sed 's/.*https/https/' dalfox_results.txt > cleandalfox.txt

```

## 🧠 Workflow Summary

```
Live Subdomains
      ↓
Katana Crawl
      ↓
Parameter Filtering
      ↓
URL Normalization
      ↓
Reflection Detection
      ↓
Noise Removal
      ↓
Dalfox Exploitation
```

---

## 🎯 Why This Works

* Reduces false positives
* Focuses on **real reflection points**
* Uses **context-aware scanning**
* Matches **bug bounty methodologies**

---

## ⚠️ Disclaimer

This project is for **educational and authorized testing only**.
The author is not responsible for misuse.

