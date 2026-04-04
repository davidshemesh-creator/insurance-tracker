# Insurance Tracker — Project Memory

> קובץ זה נקרא אוטומטית בכל שיחה שנפתחת מתיקייה זו. הוא מחליף בריפינג — לא צריך להסביר מחדש.

---

## מה הפרויקט הזה

אפליקציית Web לניהול פוליסות ביטוח וחסכון אישיות — כרגע של דוד.

**קובץ יחיד:** `index.html` — כל ה-HTML, CSS, JavaScript בקובץ אחד.
**אחסון:** localStorage בלבד, אין שרת, אין DB.
**שרת dev:** `server.js` (Node.js) על פורט 8080.

---

## ארכיטקטורה

### נתונים ב-localStorage
```
KEYS.policies  → מערך פוליסות
KEYS.payments  → מערך תשלומים בפועל
```

### סוגי פוליסות
- **ביטוחים:** בריאות, חיים, סיעודי, רכב, דירה, נסיעות, אחר
- **חסכון וצבירה:** פנסיה, קרן השתלמות, קופת גמל, חסכון
- `SAVING_TYPES = ['פנסיה','קרן השתלמות','קופת גמל','חסכון']`

### תדירויות תשלום
`monthly | quarterly | semi-annual | annual | one-time`
הפונקציה `toMonthly(amount, frequency)` ממירה לסכום חודשי.

---

## מבנה ה-UI (דפים)

| ID | שם | תוכן |
|----|----|------|
| `#page-dashboard` | Dashboard | לוח בקרה ראשי |
| `#page-policies` | פוליסות | רשימת כל הפוליסות + הוספה |
| `#page-monthly` | סיכום חודשי | טבלת 12 חודשים |
| `#page-payments` | תשלומים | היסטוריית תשלומים |
| `#page-settings` | הגדרות | ייבוא/יצוא CSV |

---

## Dashboard — מצב נוכחי (פברואר 2026)

### Layout: 3 עמודות (RTL)
```
grid-template-columns: 320px  minmax(0,1fr)  210px
                       ימין   אמצע           שמאל
                       [פאי]  [תשלומים]     [שנתי]
```

**חשוב — RTL Grid:** ב-`dir="rtl"`, ילד **ראשון** ב-HTML = עמודה **ימנית** ב-grid. סדר ה-HTML: פאי → תשלומים → שנתי.

### עמודה ימנית (פאי)
- קוביית סה"כ חודשי (ביטוחים + חסכון) עם border ורוד
- פאי ביטוחים — SVG `#donut-svg-ins`, legend `#donut-legend-ins`
- פאי חסכון — SVG `#donut-svg-sav`, legend `#donut-legend-sav`
- כותרת כל פאי: ימין = שם הפאי, שמאל = "X פוליסות פעילות"
- **אין** קוביות מתחת לפאי — הסכום מוצג גדול **בתוך** הפאי

### עמודה אמצעית (תשלומים)
- `.monthly-checklist-wrap` — רשימת תשלומי החודש הנבחר
- בורר חודש/שנה (`#donut-sel-month`, `#donut-sel-year`) נמצא **בעמודת הפאי** מעל הקוביה

### עמודה שמאלית (סיכום שנתי) — עודכנה 2026-03-03
- **ראש עמודה:** קוביית "🏦 צבירה כוללת" עם `border: 1px solid var(--accent2)`, צבע טקסט `var(--accent2)` (סגול)
  - זהי הפרדה בין "stock" (מה שצבור) לבין "flow" (תשלומים שוטפים)
- **כותרת:** `📊 סיכום שנתי` — מתחת לצבירה, מיושר לימין
- **3 קוביות נוספות:** תשלומים בפועל השנה, ביטוחים השנה, חסכון השנה
- כל הנתונים = **בפועל בלבד** (ינואר עד החודש הנוכחי)

---

## פונקציות מפתח

| פונקציה | מה עושה |
|---------|---------|
| `renderDashboard()` | מרנדר את כל ה-dashboard |
| `renderDonut()` | מחשב ומציג את שני הפאי-ים + כל ה-stat cards |
| `drawDonut(data, total, isFuture, svgId, legendId, emptyLabel)` | SVG טהור — מצייר פאי בודד |
| `donutSelChange()` | handler לשינוי חודש/שנה |
| `initDonutSelects()` | מאתחל את ה-selectors לחודש/שנה הנוכחיים |
| `isActive(p)` | בודק אם פוליסה פעילה היום |
| `renderMonthlyChecklistForMonth(year, month)` | מציג רשימת תשלומים לחודש |
| `toggleMonthlyPaid(policyId, alreadyPaid, existingPayId)` | מסמן/מבטל תשלום |
| `renderMonthlyTable()` | טבלת 12 חודשים עם footer |
| `toMonthly(amount, frequency)` | ממיר לסכום חודשי |

---

## Bugs שתוקנו — כדי לא לחזור עליהם

1. **`isActive()` — start=today נכשל**
   תיקון: `new Date(p.start+'T00:00:00') <= now` (לא T12:00:00)

2. **Footer טבלה — הציג מצטבר במקום חודשי**
   תיקון: `actualMonthTotals[mIdx]` ישירות, בלי `cumulative +=`

3. **CSS Grid ב-RTL — סדר עמודות הפוך**
   עיקרון: ב-RTL, ילד ראשון ב-HTML = עמודה ימנית. לכן `320px 1fr 210px` = ימין/אמצע/שמאל.

4. **donut-wrap נחתך**
   הסיבה: SVG גדול מהעמודה. תיקון: `width="100%"` על ה-SVG.

5. **`align-items: center` על flex container מצמצם ילדים**
   תיקון: `align-items: stretch` + `align-self: center` על SVG/legend.

---

## Summary Bar (שורה תחתונה) — נוסף 2026-03-02

פס אופקי בראש ה-dashboard (מעל ה-dash-grid), 5 שדות:

| ID | תוכן |
|----|------|
| `sb-monthly` | תשלום חודשי (accent ורוד) |
| `sb-balance` | צבירה כוללת (סה"כ balance כל הפוליסות) |
| `sb-annual` | עלות שנתית (monthly × 12) |
| `sb-ins` | ביטוחים (monthly) |
| `sb-sav` | חסכון (monthly) |

מתעדכן ב-`renderDonut()` — דינמי עם שינוי חודש.
CSS class: `.summary-bar`, `.summary-bar-item`, `.summary-bar-value.accent`

---

## Monthly Matching / CSV Import — עודכן 2026-03-03

### ארכיטקטורה
- כפתור "📥 ייבוא" ב-monthly-checklist-header → navigate('import', null)
- `matchPolicy(desc)` — score-based matching לשם חברה, סוג, מספר פוליסה
- COMPANY_ALIASES כוללת: מנורה, הפניקס, מגדל, הראל, אלטשולר שחם, כלל, איילון, ביטוח ישיר, שירביט
- **Filter נכון:** `policies.filter(p => !p.expiry || new Date(p.expiry) >= new Date())` — **לא** לפי `status` (שדה לא קיים!)
- Workflow: `T-tools/03-workflows/insurance-monthly-check.md`

### פונקציות מפתח

| פונקציה | מה עושה |
|---------|---------|
| `parseIsraeliDate(val)` | ממיר DD/MM/YYYY, DD.MM.YYYY, ISO → ISO string |
| `matchPolicy(desc, amount)` | score-based: company aliases, type keywords, policy number |
| `renderCSVPreview()` | מציג טבלה: matched-first, custom pink checkbox, text-only לשורות מזוהות |
| `importSelectedCSV()` | שומר ל-payments, קורא renderMonthlySummary, toast עם חודשים |
| `csvRowPolicyChange(i, sel)` | עדכון match + checkbox + CSS class (ללא opacity!) |

### Chrome GPU Crash Prevention — כללים קריטיים

| ❌ אל תעשה | ✅ עשה במקום |
|-----------|------------|
| `tr.style.opacity = 0.5` | `tr.classList.toggle('csv-unmatched', !sel.value)` |
| `overflow-y: auto` על inner scroll container | הסר inner overflow — רק ה-outer גולל |
| `<select>` לכל שורה (כולל מזוהות) | `<select>` רק לשורות לא-מזוהות; מזוהות = טקסט |
| `innerHTML` + `checked` / `selected` attr | אחרי `innerHTML`: `cb.checked = true`, `sel.value = id` ב-JS |

### renderCSVPreview — עקרון מפתח
- שורות מסודרות: `matched-first` לפני render (sort on `_i` index)
- checkbox מזוהה: CSS class `.csv-cb-matched` עם `appearance: none` — ורוד + וי לבן
- אחרי `innerHTML`: לסט `cb.checked` ו-`sel.value` ב-JS loop נפרד (HTML attrs לא אמינים)

### importSelectedCSV — עקרון מפתח
- שורות מזוהות אין להן `<select>` → `const pid = r.match?.id || sel?.value || ''`
- **Dedup:** לפני push — `payments.some(pay => pay.policyId===pid && pay.date===r.date && Math.abs(pay.amount-r.amount)<0.5)`
- אם כפילות → `skipped++`, toast עם הסבר
- לאחר save: קרא `renderHistory()`, `renderDashboard()`, `renderMonthlySummary()`
- Toast: מציין חודשים ספציפיים + מספר כפילויות שדולגו

### renderMonthlyChecklistCore — עקרון מפתח (עודכן 2026-03-04)
- `paymentsThisMonth` ו-`paidThisMonth` מחושבים **לפני** `dueThisMonth`
- `dueThisMonth` כולל תמיד פוליסות עם `paidThisMonth.has(p.id)` — גם אם לא "due" לפי תדירות
- `displayAmt` = `getActualPaid(p)` כשיש תשלום, לא `toMonthly()`
- `totalPaid` = סכום תשלומים בפועל, לא הערכה

### matchPolicy(desc, amount) — עקרון מפתח (עודכן 2026-03-04)
- מקבל `amount` מה-CSV כפרמטר שני
- אם יש score > 0 ו-`amount > 0`: מוסיף ניקוד קרבת סכום vs `toMonthly(p.amount, p.frequency)`
  - within 5%: +10 | within 15%: +6 | within 30%: +3
- פותר בעיית disambiguation כשיש 5 פוליסות מנורה שונות

### deduplicatePayments() — פונקציה חדשה (2026-03-04)
- מנקה כפילויות מ-payments (same policyId + date + amount)
- נקרא מהגדרות עם כפתור "נקה כפילויות"
- `renderSettings()` מציג count של כפילויות שנמצאו

---

## Gmail Integration (נוסף 2026-03-03)

### ארכיטקטורה
- **חיבור:** OAuth 2.0 דרך `server.js` — לא דרך הדפדפן ישירות
- **credentials.json** — זהות האפליקציה (מ-Google Cloud Console), לא נכנס לgit
- **token.json** — נוצר אחרי אימות המשתמש, נשמר מקומית, לא נכנס לgit

### Scopes (הרשאות בלבד)
- `gmail.readonly` — קריאת מיילים
- `gmail.send` — שליחת מיילים
- **אין:** delete, modify, labels

### API Endpoints בserver.js
| Route | Method | תפקיד |
|-------|--------|--------|
| `/api/gmail/auth` | GET | מפנה ל-Google OAuth |
| `/api/gmail/callback` | GET | מקבל token, שומר token.json |
| `/api/gmail/status` | GET | מחובר? + כתובת מייל |
| `/api/gmail/disconnect` | GET | מוחק token.json |
| `/api/gmail/messages` | GET | רשימת מיילים (query param: `q`, `max`) |
| `/api/gmail/message/:id` | GET | מייל מלא (body מפוענח מbase64) |
| `/api/gmail/send` | POST | שליחת מייל (`to`, `subject`, `message`) |

### UI ב-Settings
- כפתור חיבור/ניתוק + סטטוס חשבון
- שדה חיפוש מיילים (ברירת מחדל: ביטוח/פנסיה)
- לחיצה על מייל → `alert()` עם תוכן מלא
- טופס שליחת מייל

### קבצים שהשתנו
- `server.js` — הוסף Gmail routes
- `index.html` — הוסף Gmail section בSettings + JS functions
- `package.json` — נוצר עם `googleapis`
- `.gitignore` — הוסף credentials.json, token.json, node_modules
- `הפעל.command` — עודכן מ-python לnode server.js

---

## מה בתכנון (שלב 3)

- גרף יתרות צבירה לאורך זמן
- התרעות אוטומטיות לפוליסות שפגות תוקפן
- דוח שנתי מסכם

---

## Session End Protocol (MANDATORY)

בסוף כל שיחת פיתוח — לפני סגירה — בצע סגירת לופ אוטומטית:

**תנאי הפעלה:** השיחה כללה שינוי קוד, תיקון באג, או החלטת עיצוב.

**מה לעדכן:**
1. `M-memory/learning-log.md` — pattern חדש, באג שתוקן, טכניקת debugging
2. `M-memory/decisions.md` — אם התקבלה החלטה ארכיטקטורית או עיצובית
3. `P-projects/_personal/insurance-tracker/spec-v1.md` — אם השתנה סטטוס / layout / data model
4. `O-output/insurance-tracker/CLAUDE.md` (קובץ זה) — אם השתנו פונקציות מפתח, bugs שתוקנו, או ארכיטקטורה

**לא לשאול** — לסגור את הלופ בפעולה. אם לא בטוח מה לכתוב — לכתוב גרסה קצרה.

---

## טון עם דוד בפרויקט הזה

- עברית ישירה, ללא "יופי נהדר"
- מספר שאלות קצרות, לא פסקאות
- כשמשהו לא עובד — מאבחן JavaScript לפני שמנחש
- מציג screenshot לאחר כל שינוי ויזואלי

---

## Firebase + GitHub Pages (נוסף 2026-04-04)

### ארכיטקטורה חדשה
- **אחסון:** Firebase Firestore (לא localStorage)
- **Auth:** Firebase Auth — Google Sign-in
- **Hosting:** GitHub Pages — `https://davidshemesh-creator.github.io/insurance-tracker/`
- **GitHub repo:** `https://github.com/davidshemesh-creator/insurance-tracker`
- **Firebase project:** `insurance-tracker-41de2`

### מבנה Firestore
```
users/{uid}/data/policies → { items: [...] }
users/{uid}/data/payments → { items: [...] }
```

### Deploy
```bash
git add index.html && git commit -m "..." && git push
```
GitHub Pages מתעדכן תוך ~2 דקות.

### Firestore Security Rules
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/{document=**} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

### פונקציות מפתח
| פונקציה | מה עושה |
|---------|---------|
| `_userDoc(sub)` | מחזיר ref ל-`users/{uid}/data/{sub}` |
| `pp()` | שומר policies ל-Firestore |
| `py()` | שומר payments ל-Firestore |
| `_loadFromFirestore()` | טוען נתונים + מיגרציה מ-localStorage |
| `signInWithGoogle()` | כניסה עם Google Popup |
| `signOutUser()` | יציאה |

*עודכן לאחרונה: 2026-04-04*
