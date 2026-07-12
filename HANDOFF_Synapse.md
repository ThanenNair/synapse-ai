# Handoff: Synapse AI — Paediatric Tools Sync
**Date:** 2026-06-29  
**Session goal:** Update all Paediatric Tools in Synapse AI to match PaedsCare Clinical Suite.

---

## Files
| File | Path |
|------|------|
| Synapse AI | `C:\Users\PC\Desktop\Synapse AI\synapse.html` (17,671 lines) |
| PaedsCare | `C:\Users\PC\Desktop\PaedsCare Clinical Suite\paedscare.html` (6,928 lines) |

---

## What Was Done This Session

### 1. BSA Calculator — COMPLETED ✅
Added Body Surface Area (Mosteller formula) to the **BP Centile tab** in Synapse.

**CSS** added after `.bppc-err` rule (~line 2179):
```css
.bsa-section{margin-top:1.1rem;background:var(--card);border:1.5px solid var(--border);border-radius:12px;overflow:hidden;}
.bsa-hdr{display:flex;align-items:center;justify-content:space-between;padding:.55rem .85rem;background:linear-gradient(90deg,rgba(44,122,123,.1),transparent);border-bottom:1.5px solid var(--border);}
.bsa-hdr-title{font-size:.78rem;font-weight:700;color:var(--primary);}
.bsa-formula-tag{font-size:.58rem;font-weight:700;text-transform:uppercase;letter-spacing:.04em;color:#2c7a7b;background:rgba(44,122,123,.1);border:1px solid rgba(44,122,123,.2);border-radius:99px;padding:.1rem .45rem;}
.bsa-form{display:grid;grid-template-columns:1fr 1fr;gap:.6rem;padding:.75rem .85rem .65rem;}
.bsa-result-card{margin:0 .85rem .85rem;background:linear-gradient(135deg,#2c7a7b,#0987a0);border-radius:10px;padding:.75rem 1rem;display:flex;align-items:center;justify-content:space-between;}
.bsa-result-label{font-size:.68rem;font-weight:600;color:rgba(255,255,255,.75);text-transform:uppercase;letter-spacing:.05em;}
.bsa-result-value{font-family:'Poppins',sans-serif;font-size:2rem;font-weight:800;color:#fff;line-height:1;}
.bsa-result-unit{font-size:.78rem;font-weight:600;color:rgba(255,255,255,.75);margin-top:.25rem;}
.bsa-formula-note{font-size:.6rem;color:rgba(255,255,255,.55);margin-top:.2rem;}
```

**HTML** added inside BP Centile tab after `<div id="bppcResult"></div>`:
```html
<div class="bsa-section">
  <div class="bsa-hdr">
    <span class="bsa-hdr-title">Body Surface Area</span>
    <span class="bsa-formula-tag">Mosteller</span>
  </div>
  <div class="bsa-form">
    <div class="bppc-field">
      <label>Weight (kg)</label>
      <input class="bppc-inp" id="bsaWt" type="number" placeholder="e.g. 12.5" min="0.5" max="250" step="0.1" oninput="bsaCalc()">
    </div>
    <div class="bppc-field">
      <label>Height (cm)</label>
      <input class="bppc-inp" id="bsaHt" type="number" placeholder="e.g. 85" min="30" max="220" step="0.5" oninput="bsaCalc()">
    </div>
  </div>
  <div id="bsaResult"></div>
</div>
```

**JS** added inside bppc IIFE before `})();` (~line 13197):
```javascript
window.bsaCalc=function(){
  var wt=parseFloat(document.getElementById('bsaWt')?.value)||0;
  var ht=parseFloat(document.getElementById('bsaHt')?.value)||0;
  var el=document.getElementById('bsaResult'); if(!el)return;
  if(!wt||!ht){el.innerHTML='';return;}
  var bsa=Math.sqrt(ht*wt/3600);
  el.innerHTML=
    '<div class="bsa-result-card">'
    +'<div><div class="bsa-result-label">BSA</div><div class="bsa-formula-note">√(H × W ÷ 3600)</div></div>'
    +'<div style="text-align:right"><div class="bsa-result-value">'+bsa.toFixed(2)+'</div><div class="bsa-result-unit">m²</div></div>'
    +'</div>';
};
```

---

### 2. Sedation Dilution Tool — COMPLETED ✅
Synapse already had the updated `sed2-*` layout, patient name input, 4-drug order (Kytril→Atropine→Midazolam→Ketamine), and `sedPrint()` from a prior session.

**Only remaining fix applied this session:** `_sedRound()` logic corrected to match PaedsCare exactly.

Old (Synapse):
```javascript
function _sedRound(val, unit) {
  var r = Math.round(val * 10) / 10;
  var disp = r % 1 === 0 ? r.toFixed(0) : r.toFixed(1);
  return '→ ' + disp + ' ' + unit;
}
```

New (matches PaedsCare):
```javascript
function _sedRound(val, unit) {
  var r = Math.round(val);
  return '→ ' + (r === 0 && val > 0 ? val.toFixed(1) : r) + ' ' + unit;
}
```

---

## Status of All Paediatric Tools in Synapse

| Tool | Status |
|------|--------|
| BP Centile Calculator | ✅ Matches PaedsCare |
| BSA Calculator (Mosteller) | ✅ Added this session |
| Sedation Dilution | ✅ Matches PaedsCare (fixed this session) |
| NIP Catch-Up Calculator | ✅ Matches PaedsCare (prior session) |
| Neonatal Jaundice / HOL Calculator | ✅ Matches PaedsCare (prior session) |
| IV Fluid Calculator (with dehydration) | ✅ Matches PaedsCare (prior session) |
| Seizure Protocol | ✅ Matches PaedsCare — 10 drugs (prior session) |
| Developmental Milestones | ✅ Present in Synapse |
| Asthma Drug Reference | ✅ Present in Synapse |

**All paediatric tools are now in sync. The Paediatric Tools sync task is COMPLETE.**

---

## Key Architecture Notes

- **Single-file app**: `synapse.html` — no build system, open directly in Chrome.
- **Dark mode**: Synapse uses `body.dark`; PaedsCare uses `[data-theme="dark"]`. When porting CSS, replace `[data-theme="dark"]` selectors with `body.dark`.
- **Paediatric tools location**: Inside `#paedsToolsOverlay` > `.paeds-tab-pane` elements, z-index: 150.
- **bppc IIFE**: `bsaCalc`, `bppcCalc`, `bppcCopy` live inside a `(function(){ ... })();` block. New functions must go before the closing `})();`.
- **CSS variables in use**: `var(--card)`, `var(--border)`, `var(--primary)`, `var(--secondary)`, `var(--text-muted)`, `var(--bg)`, `var(--grad-primary)`, `var(--accent)`.

---

## Next Steps / Nothing Pending

There are no known outstanding differences between Synapse paediatric tools and PaedsCare. If the user identifies a new mismatch, compare the relevant section in both files directly.

To continue: open `synapse.html` in Claude Code on the other PC and describe what to work on next.
