# Insurance Tracker — Project Memory

> קובץ זה נקרא אוטומטית בכל שיחה שנפתחת מתיקייה זו. הוא מחליף בריפינג — לא צריך להסביר מחדש.

---

## ⭐ Latest Redesign — 2026-05-19 (Design DNA from claude.ai)

**מצב נוכחי (יציב):** Dashboard + Policies + Popup הוטמעו מחדש בשפה ויזואלית אחידה ע"פ design מ-`claude.ai/design` (oklch warm-purple, Heebo, magenta accent, hue-coded per category). זה הסגנון העכשווי — סקציות הישנות בקובץ הזה (תחת "אפריל 2026") **לא רלוונטיות יותר**.

### Design Tokens — מוגדרים על `#page-dashboard, #page-policies`
```css
--d-bg:        oklch(0.16 0.012 320);   /* dark warm-purple base */
--d-surf:      oklch(0.215 0.013 320);
--d-line:      oklch(0.34 0.013 320);
--d-fg:        oklch(0.97 0.005 320);
--d-fg-mut:    oklch(0.68 0.012 320);
--d-fg-dim:    oklch(0.5 0.01 320);
--d-accent:    oklch(0.72 0.22 350);    /* magenta */
--d-green:     oklch(0.78 0.18 150);
```
**Font:** Heebo (Google Fonts) + tabular-nums lining-nums on `.dash3-money`/`.pc-money`.

### Category Hue Mapping — Single Source of Truth
```js
DASH3_CATS = {
  'בריאות':      { hue: 12,  icon: 'heart'   },  // red
  'חיים':        { hue: 230, icon: 'shield'  },  // blue
  'סיעודי':      { hue: 280, icon: 'helix'   },  // purple
  'רכב':         { hue: 55,  icon: 'car'     },  // orange (was 38, moved 2026-05-19 to differentiate from health)
  'דירה':        { hue: 152, icon: 'home'    },  // green
  'נסיעות':      { hue: 60,  icon: 'shield'  },
  'אחר':         { hue: 200, icon: 'shield'  },
  'פנסיה':       { hue: 312, icon: 'pension' },  // magenta
  'קרן השתלמות': { hue: 88,  icon: 'study'   },  // yellow-green
  'קופת גמל':    { hue: 195, icon: 'gem'     },  // teal
  'חסכון':       { hue: 0,   icon: 'piggy'   },
};
DASH3_ICONS // inline SVGs (Lucide-style)
dash3CategoryOf(p) // returns cat object for any policy
```
Used **everywhere**: dashboard rows, popup header (gradient), policy cards (gradient + border + inline `--hue`).

### Dashboard v3 — `#page-dashboard`
**Layout (1440px max):** topbar → 2 summary cards (1.45fr / 1.25fr) → 2 sections (ביטוחים / חיסכון) with policy rows.

- **Topbar:** brand icon + month switcher (`dash3PrevMonth/dash3NextMonth`, chevrons: prev=right-pointing, next=left-pointing in RTL) + search + status pulse. **No "+ הוסף פוליסה"** (lives only on policies page).
- **Card 1 — "תשלום החודש":** big number = **paid actual** (not "remaining"), eyebrow "שולם בחודש זה", ring = paid/due ratio, footer chips = unpaid policies. **Overage logic:** if `paidAmountTotal > insMonthlyTotal + ₪50` → show "שולם ₪X יותר מההערכה" (neutral if ≤10% over, **red** if >10% — index variance tolerance).
- **Card 2 — "צבירה כוללת":** big balance + monthly deposits + breakdown chips per savings sub-type.
- **Sections:** rows with category icon · tag+insured · company+# · **12 trail dots (Jan-Dec of dash3Year, current month = highlighted)** · monthly amount · balance+liquidity (savings) · paid toggle (⚡/✓).
- **Per-month accuracy:** uses `dueAmount(p)` + `dash3IsDueInMonth(p,y,m)` — **NOT** averaged `toMonthly()`. So יולי may show ₪1,716 (includes car installment) while מאי shows ₪1,116.
- **Trail dots:** calendar-year, not rolling. Future months (after today) = dashed empty circle.
- **State:** `dash3Year`, `dash3Month` globals; `dash3TogglePaid(id)` creates/deletes a payment for current view-month.

### Policies Page v2 — `#page-policies`
**Layout (1440px max):** topbar → grouped sections.

- **Topbar:** page-icon + "פוליסות / X פעילות · Y פגות" + segment (פעילות/פגות תוקף) · search (min-width 200px) · `<select class="group-select pc-group-select">` (width 110px, "לפי סוג / לפי חברה") · **"+ הוסף פוליסה" button** (`.dash3-btn-primary`, white-space:nowrap, flex-shrink:0).
- **Grid:** `repeat(3, 1fr)` desktop · 2 cols ≤1100px · 1 col ≤700px.
- **Card structure:** header (icon + tag/company + 📎 clip placeholder) → stat box (big amount + label, border-top/bottom soft) → people line (מבוטחים/סוכן) → chips (פעיל/פוגת/נזיל). **`--hue` inline per card** drives the gradient + border.
- **By-type mode (default):** 2 sections (ביטוחים/חסכון) with sub-section per type (כל סוג מקבל header של אייקון+שם+ספירה+סטטיסטיקה ב-`/שנה`). **Gap בין סוגי-משנה: 60px** (לא 18 — דחוס מדי).
- **By-company mode:** sections per company with `.pc-company-logo` (white square + img) or fallback letter. Companies with logos sort first.
- **Sub-section sort:** קרן השתלמות ממוינת לפי `liquidityDate` עולה (כבר נזיל → קרוב → רחוק → ללא תאריך).
- **Annual totals (not monthly):** policies page has no month context, so all sums are **`annualAmount(p)`** = `amount × {12, 4, 2, 1}` by frequency. Label: "עלות שנתית" / "הפקדה שנתית" / "X ₪/שנה" — never "חודשי" here.

### Popup (view-modal) — Policy Details
- **Header:** colored gradient by category hue (`linear-gradient(135deg, oklch(0.34 0.13 H), oklch(0.22 0.05 H))`) · SVG cat-icon (oklch tinted box) · title (`p.tag || p.type`) · sub (`company · # · agent`) · big number on left (balance for savings, amount+frequency for insurance).
- **Edit pencil icon** (`.popup-edit-btn` Lucide pencil SVG) in the `.popup-section-title-row` next to "פרטי הפוליסה" (left side in RTL). **No more floating top-right edit emoji.**
- **Detail cards:** 2-col grid. Row 1: עלות חודשית | סטטוס. Row 2: תחילת תוקף | תאריך פקיעה. Row 3+: optional (נזילות, צבירה עדכנית ל-).
- **Documents section:** dropzone placeholder calling `uploadPolicyDoc(id)` → toast "בפיתוח".
- **Payments section (collapsed by default — `display: none` on rows):** summary line (`X תשלומים ב-{currentYear} · סה"כ ₪Y · אחרון DATE`) + 2 action buttons inline: `+ הוסף תשלום` + `ראה הכל (N)`. Expanded: **max-height 160px scrollable list** + year tabs (`.payments-tabs` shown only if multiple years exist; clicks `filterPaymentsByYear(year)`). Pay row = 4-col grid (date · source · amount · 🗑).

### Key Helpers Added (2026-05-19)
| פונקציה | מה עושה |
|---|---|
| `dash3CategoryOf(p)` | מחזיר `{he, hue, icon}` לפי `p.type` |
| `DASH3_ICONS[name]` | מחרוזת SVG inline (heart, shield, car, ...). **Heart rewritten 2026-05-19** ל-Lucide path נקי. |
| `dash3PolicyHistory(p, year)` | 12 ערכים (ינואר-דצמבר) של 1/0 לפי `payments` |
| `dash3IsDueInMonth(p,y,m)` | האם הפוליסה אמורה להיגבות בחודש נתון (לפי `paymentMonths`/frequency). **תמיד false לחסכון.** |
| `dash3NextDueLabel(p,y,m)` | "תשלום הבא: יום X" / "אוק׳ YYYY" |
| `dash3TogglePaid(id)` | מוסיף/מסיר payment ל-`dash3Year`/`dash3Month` |
| `annualAmount(p)` *(inline ב-renderPolicies)* | `amount × {12,4,2,1}` |
| `editFromView()` | קורא ל-`openEditPolicy(viewModalPolicyId)` |
| `togglePolicyPayments(btn)` | מצב מקופל/פתוח של רשימת תשלומים בpopup |
| `filterPaymentsByYear(y)` | סינון פנימי של רשימת התשלומים בpopup לפי טאב שנה |
| `uploadPolicyDoc(id)` | placeholder — מציג toast "בפיתוח" |

### Critical Rules — Don't Break
1. **`--d-*` variables must be defined on every page that uses them.** Today's bug: `#page-policies` was missing the var definitions → button looked dead. Fix: declarations on `#page-dashboard, #page-policies {}`.
2. **`--hue` inline on each card** drives ALL color (background gradient + border + icon tint). Never hard-code per-type colors elsewhere.
3. **`white-space: nowrap` + `flex-shrink: 0`** on `.dash3-btn-primary` — without it the button wrapped to 2 lines when topbar got crowded.
4. **Dashboard = per-month actual (`dueAmount` + `isDueInMonth`); Policies page = annual total (`annualAmount`).** Never use `toMonthly` for cash-flow displays — it averages and lies about July vs February for car insurance.
5. **SAVING_TYPES still always return `false` from `dash3IsDueInMonth`** — savings don't have a fixed schedule. For sub/total calculations, savings monthly amount comes from `p.amount > 0 ? toMonthly(...) : 0` (averaged is fine for savings — usually they ARE monthly).
6. **RTL chevrons:** prev button (visual right in RTL) = SVG points right (no rotation). Next button (visual left) = `transform: rotate(180deg)`. Reverse of LTR intuition.

### Dead Code (Safe to Remove Eventually)
- `renderDonut()`, `donutSelChange()`, `initDonutSelects()`, `donutYear`/`donutMonth` globals — orphaned after dashboard rewrite. Still defined but unreferenced from the new dashboard.
- `renderMonthlyChecklistForMonth()`, `renderMonthlyChecklist()`, `renderMonthlyChecklistCore()` — same.
- `toggleMonthlyPaid()` — replaced by `dash3TogglePaid()` for the new dashboard.
- Old policies-page CSS classes (`.policies-grid`, `.policies-col`, `.ptable-*`, `.anchor-*`, `.policies-missing-tag-banner`) — replaced by `.pc-*`.
- `openMissingTagsModal()` / `saveMissingTags()` / `#missing-tags-modal` — the missing-tag banner was removed from the new policies page. Functions still exist; modal HTML still in DOM; reachable only if some old code path calls `openMissingTagsModal()`. Leave as-is for now.

---

## מה הפרויקט הזה

אפליקציית Web לניהול פוליסות ביטוח וחסכון אישיות — כרגע של דוד.

**קובץ יחיד:** `index.html` — כל ה-HTML, CSS, JavaScript בקובץ אחד.
**אחסון:** Firebase Firestore + Google Auth.
**שרת dev:** `node server.js` מתיקיית הפרויקט → `http://localhost:8080`
**GitHub Pages (פרודקשן):** `https://davidshemesh-creator.github.io/insurance-tracker/`

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

**שדות מיוחדים לפוליסה שנתית:**
- `installments: number` — כמה תשלומים בשנה (ברירת מחדל: 1)
- `paymentMonths: number[]` — אילו חודשים ספציפית (1=ינואר...12=דצמבר)
- דוגמה: רכב בתשלום 3 חודשים: `installments: 3, paymentMonths: [1,2,3]`

**פונקציות חישוב — הבחנה קריטית:**
- `toMonthly(amount, freq)` — ממוצע חודשי לתצוגה בלבד (לא ל-"כמה יורד החודש")
- `dueAmount(p)` — מה שמשלמים בפועל בכל תשלום (`amount / installments`)
- `isDueInMonth(p, year, month)` — האם הפוליסה בתשלום בחודש נתון

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

## Policies Page — מצב נוכחי (אפריל 2026)

- **Layout:** 2 עמודות — ביטוחים (ימין) | חסכונות לא נזילים (שמאל)
- **CSS:** `.policies-grid { gap: 20px }` + `.policies-col { background: #1a1a1a; border-radius: 14px; padding: 20px 18px 24px; }` — ללא border
- **כותרת עמודה:** `ביטוחים (N)` — מספר פוליסות ליד השם, סכום חודשי בצד השני
- **padding הדף:** `36px 44px`
- **היררכיה בכרטיס:** סוג (גדול) → tag → חברה·סוכן·מספר (שורה אחת)
- **צבעים:** לפי סוג בלבד (`TYPE_COLORS`) — לא לפי חברה

## Dashboard — מצב נוכחי (אפריל 2026)

### קוביית "שולם החודש" — לוגיקה נכונה
- **המספר הגדול:** `actualPaidThisMonth` = תשלומים מאושרים בלבד (ללא state)
- **שורה מתחת:** `משוער: ₪X · נותר ~₪Y` או `+₪Y מעל ההערכה` (ניטרלי, לא אדום)
- **הערכה:** `getEstimated(p)` — לפוליסה שנתית: `dueAmount(p)` רק בחודשים שב-`paymentMonths`
- **חודשים עתידיים:** מציג הערכה בלבד, כותרת "💳 הערכת תשלומים"

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
| `toMonthly(amount, frequency)` | ממוצע חודשי — לתצוגה בלבד |
| `dueAmount(p)` | סכום בפועל לתשלום (amount / installments לפוליסה שנתית) |
| `isDueInMonth(p, year, month)` | האם הפוליסה בתשלום בחודש נתון — משתמש ב-paymentMonths |
| `renderHistory()` | היסטוריית תשלומים מקובצת לפי חודש עם headers, פילטרים: policy/year/month |
| `applyDefaultPolicy(sel)` | ממפה את כל שורות ה-CSV שלא הותאמו לפוליסה שנבחרה |
| `histYearChange()` | מאפס פילטר חודש בעת שינוי שנה בהיסטוריה |
| `toggleInstallments(existingMonths)` | מציג/מסתיר שדות installments + בונה month dropdowns דינמי |

---

## Bugs שתוקנו — כדי לא לחזור עליהם

14. **SAVING_TYPES + isDueInMonth — באג שחזר 3 פעמים (2026-04-06)**
    הסיבה: `isDueInMonth` מחזיר תוצאות שגויות לפוליסות חסכון (אין להן לוח זמנים קבוע).
    תיקון כפול:
    - `renderHistory` expectedCount: מסנן SAVING_TYPES מ-isDueInMonth + מוסיף `savingsPaidCount` בנפרד
    - Split bar בלוח בקרה: `savPolsDue = savPolsPaid` (חסכונות = רק מה שנשלם בפועל)
    **כלל:** בכל שימוש ב-isDueInMonth — לבדוק קודם `!SAVING_TYPES_SET.has(p.type)`

15. **State payments בספירת חסכונות (2026-04-06)**
    הסיבה: `paidThisMonthIds` סינן `paidBy==='state'` — אך הפקדות מעביד לפנסיה/קרן השתלמות הן state.
    תיקון: `paidThisMonthIdsSav` — Set נפרד ללא סינון state, רק לחסכונות.

16. **Progress bar = count-based (2026-04-06)**
    הסיבה: amount-based bar לא מגיע ל-100% כי סכומים משתנים בין חודשים.
    תיקון: `insPct = insPolsPaid.length / insPolsDue.length * 100` — מגיע ל-100% כשכל הפוליסות שולמו.
    תווית: `צפוי/שולם` (X/Y) — לא אחוזים. עודף: ספרת שולם בצהוב.

17. **Annual bar = time-based (2026-04-06)**
    פברואר = 2/12, דצמבר = מלא. לא תלוי בסכומים. מציג שולם בפועל בלבד (ללא "צפי").
    שנת התצוגה מוצגת בכותרת (`id="dash-annual-year"`).

13. **Monthly summary — רווח ענק משמאל לטבלה (2026-04-06)**
    הסיבה: `table { width: auto }` ב-RTL — הטבלה מצמצמת לתוכן ומיושרת ימינה, מותירה רווח ריק ב-container.
    תיקון: `width: 100%` + `table-layout: fixed` על `.monthly-table`. רוחב עמודה ראשונה (`140px`) ואחרונה (`70px`) מוגדרים ב-CSS. הוסרו `min-width`/`max-width` מ-inline styles ב-JS.

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

6. **Annual summary filter — Invalid Date על פוליסות בלי expiry**
   תיקון: `const e = p.expiry ? new Date(p.expiry+'T12:00:00') : null` — null check חובה לפני כל שימוש בתאריך תפוגה.

7. **CSV import — שורות סה"כ נכנסות כתשלומים**
   תיקון: לדלג על שורות שבהן `dateC >= 0` אבל `parseIsraeliDate` מחזיר ריק.

8. **Firestore silent failures**
   תיקון: הוסף `.catch(e => console.error(...))` ל-`pp()` ו-`py()` + קריאה ל-`db.enablePersistence()` לפני כל גישה אחרת.

9. **Dashboard מציג מספר שגוי — ערבוב actual + estimated**
   הבעיה: `getMonthly()` החזיר `actualAmount` לפוליסות ששולמו + `toMonthly()` לשאר → מספר שלא תואם שום view אחר.
   תיקון: הפרדה מוחלטת — `actualPaidThisMonth` = תשלומים בלבד, `estimatedAll` = toMonthly() בלבד.

10. **Annual installments — interval אוטומטי לא עובד בביטוח ישראלי**
    הבעיה: ביטוח רכב בתשלומים = לא כל 6 חודשים, אלא 3 חודשים עוקבים ואז נגמר.
    תיקון: `paymentMonths: [1,2,3]` — המשתמש בוחר חודשים ספציפיים, לא interval.

11. **CSS inline style גובר על class**
    הבעיה: `tr.classList.toggle('csv-unmatched')` לא עובד כשיש `tr.style.opacity = '0.4'` קשיח.
    תיקון: לאפס גם inline style: `tr.style.opacity = sel.value ? '' : '0.4'`

12. **Company colors הוסרו לגמרי (2026-04-06)**
    צבעים לפי TYPE בלבד (`TYPE_COLORS`). הוסרו: `companyColor()`, `COMPANY_COLORS_DEFAULT`, picker בSettings.
    כל שימוש ב-`companyColor()` הוחלף ב-`typeColor(p.type)`.

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

## State Payments (הפקדת מדינה)

- **שדה:** `payment.paidBy = 'state'`
- **CSV import:** צ'קבוקס "🏛️ הפקדת מדינה" + dropdown "מפה הכל ל..." לפוליסה ספציפית
- **Monthly checklist:** מציג 🏛️ במקום ✓ — נפרד מ-`totalPaid` של המשתמש
- **History:** מציג tag של 🏛️ מדינה בשורת התשלום
- **Annual summary:** נכלל בסכומים בפועל (actual totals)

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

---

## Related

[[the-system-v8/P-projects/_personal/insurance-tracker/project-brief|Insurance Tracker Brief]] · [[the-system-v8/A-agents/developer-agent|Eli (Developer)]]
