# SAYN — Instagram Follower Analyzer

> Sizi kim izləmir, kim yeni gəldi, kim getdi — hamısını görün. Sıfır server, sıfır hesab, 100% brauzer daxilində.

**Versiya:** 3.6.5 · **Manifest:** V3 · **Platform:** Chrome Extension

---

## 📌 Qısa Baxış

SAYN — Instagram üçün izləyici analitika extension-udur. Extension heç bir backend-ə malik deyil; bütün hesablama və məlumat saxlanması yalnız istifadəçinin öz brauzerində (`localStorage`) baş verir.

Analiz üçün **iki müstəqil, tamamlayıcı üsul** dəstəklənir:

| Üsul | Necə işləyir | Nə vaxt istifadə edilir |
|---|---|---|
| **Live Scan** | Instagram-ın öz daxili (private) API-larına, istifadəçinin öz sessiya cookie-si ilə, birbaşa brauzerdən sorğu göndərilir. | Standart, tam avtomatik axın. |
| **Data İdxalı** | Instagram-ın rəsmi "Download Your Information" JSON eksportu birbaşa parse edilir — heç bir şəbəkə sorğusu yoxdur. | Live Scan Instagram tərəfindən müvəqqəti bloklandıqda (401/rate-limit) ehtiyat üsul kimi. |

Hər iki üsulun nəticəsi eyni analiz mühərrikindən (`sayn-data.js`) keçir, ona görə Not Following Back, Yeni/İtirilmiş İzləyicilər, Top Fans və digər bütün hesablamalar hər iki halda eyni şəkildə işləyir.

---

## ✨ Xüsusiyyətlər

- **Not Following Back** — sizi izləməyən, amma siz izlədiyiniz hesablar
- **Yeni & İtirilmiş İzləyicilər** — skan tarixçəsi müqayisəsi, istifadəçi adına görə etibarlı diff
- **Bütün Bloklananlar** — tam blok siyahısı (API + fallback)
- **Story Gizlədilənlər** — story-nizi kimlərdən gizlətdiyiniz (4 fallback endpoint)
- **Top Fans** — son postlardakı şərhlərə görə ən aktiv izləyicilər
- **Ghost Followers & İnaktiv Following** — hər hesab üçün real, canlı post-sayı yoxlaması ilə
- **Story Baxanlar** — aktiv story-lərə kim baxıb, hər story üzrə ayrıca
- **Təhlükəsiz Unfollow** — gecikmə, gündəlik limit və batch fasilələri ilə qorunan unfollow axını
- **CSV / JSON Export** — bütün nəticələri endirin
- **Rəsmi Data İdxalı** — Instagram-ın öz eksport faylından sıfır-risk analiz

---

## 🧱 Arxitektura

```
toolbar ikonuna klik
        │
        ▼
background.js — instagram.com tab-ı yoxlanır
        │
        ├─ artıq yüklənibsə → panel yoxdursa remount, varsa heç nə
        │
        └─ ilk dəfədirsə → modullar ardıcıl inject edilir:
             1. sayn-core.js     — qlobal state, storage key-lər, ayarlar
             2. sayn-styles.js   — panel CSS-i
             3. sayn-data.js     — analiz mühərriki (NFB, diff, sort, export)
             4. sayn-parser.js   — Instagram data eksport parser
             5. sayn-fetch.js    — Instagram-ın private API qatı (Live Scan)
             6. sayn-ui.js       — panel UI, naviqasiya, görünüşlər
```

`popup.html` / `popup.js` toolbar-dan sürətli statistikanı göstərir və panelə naviqasiya təmin edir (Not Back, Yeni, İtirilmiş, Ghost, Story Viewers, Top Fans, Blocked, Hidden Story, Statistika).

### Live Scan detalları

- Following/Followers səhifələnərək (35 istifadəçi/səhifə) yüklənir.
- Instagram-ın `X-IG-WWW-Claim` anti-abuse tokeni tutulur və avtomatik saxlanaraq növbəti sorğulara əlavə edilir (401 xətalarının qarşısını almaq üçün).
- Ghost Followers və İnaktiv Following hər hesab üçün ayrı-ayrı, canlı post-sayı sorğusu ilə hesablanır (limitli say, `inactiveCheckCap`).

### Data İdxalı detalları

- Instagram Ayarlar → Hesab Mərkəzi → "Öz Məlumatınızı Yükləyin" bölməsindən JSON formatında eksport edilir.
- `followers_1.json`, `following.json`, (opsional) `blocked_accounts.json` faylları fayl adına görə avtomatik tanınır.
- Köhnə ID sxemi (timestamp-əsaslı) yeni versiyada istifadəçi adı (username) əsaslı sxemə keçib — bu, Yeni/İtirilmiş İzləyicilər hesablamasını daha etibarlı edir.

---

## 🔐 Məxfilik

- Heç bir backend, heç bir hesab qeydiyyatı yoxdur.
- Bütün məlumat yalnız `instagram.com` mənşəyinin `localStorage`-ində saxlanır.
- Extension yalnız 3 icazə tələb edir: `activeTab`, `scripting`, `tabs`.
- Extension yalnız `instagram.com` və `i.instagram.com` üzərində işləyir (`host_permissions`).

---

## 🗄️ localStorage Açarları

| Açar | Təyinat |
|---|---|
| `sayn_history_v1` | Son skan snapshotları (maks. 20) |
| `sayn_blocked_all_v1` | Blok siyahısı cache-i |
| `sayn_hidden_story_v1` | Story gizlədilmiş istifadəçilər cache-i |
| `sayn_settings_v1` | Ayarlar (gecikmə, batch, unfollow limitləri) |
| `sayn_scanning_v1` | Davam edən skan statusu |
| `sayn_unfollow_log_v1` | Son 500 unfollow qeydiyyatı |
| `sayn_top_fans_v1` | Top Fans xal cache-i |
| `sayn_ghosts_v1` | Ghost Followers nəticə cache-i |
| `sayn_following_snap` / `sayn_followers_snap` | Növbəti skan üçün "əvvəlki" snapshot |
| `sayn_www_claim_v1` | Instagram anti-abuse tokeni |
| `sayn_pk_scheme_v2` | Bir dəfəlik ID sxemi miqrasiya bayrağı |

---

## ⚙️ Ayarlar (Default Dəyərlər)

| Parametr | Dəyər | Təyinat |
|---|---|---|
| `delay` | 2500 ms | Sorğular arasında əsas gecikmə |
| `delayJitter` | +0–900 ms | Təsadüfi əlavə gecikmə |
| `batchSize` | 35 | Səhifə başına istifadəçi sayı |
| `unfollowDelay` | ~15 000 ms | İki unfollow arasında gecikmə |
| `unfollowDailyCap` | 60 | Tövsiyə olunan gündəlik unfollow limiti |
| `unfollowBatchPause` | 25 əməliyyat | Bu saydan sonra məcburi fasilə |
| `unfollowBatchCooldownMs` | 120 000 ms (2 dəq.) | Batch fasiləsinin uzunluğu |
| `ghostDays` | 90 gün | İnaktivlik hesablama üçün gün ölçüsü |
| `inactiveCheckCap` | 150 | İnaktiv Following yoxlamasında maks. hesab sayı |

Bu dəyərlər Ayarlar panelindən dəyişdirilə bilər; defaultlar Instagram-ın avtomatlaşdırma müdafiəsini tetiklməmək üçün konservativ seçilib.

---

## 🚀 Quraşdırma (Developer Mode)

1. Extension fayllarını ZIP şəklində endirin və çıxarın.
2. Chrome-da `chrome://extensions/` səhifəsini açın.
3. Sağ yuxarı küncdəki **Developer mode**-u aktivləşdirin.
4. **Load unpacked** düyməsinə basıb çıxarılmış qovluğu seçin.
5. `instagram.com`-a keçin, toolbar-dakı SAYN ikonuna basın.

> Chrome Web Store versiyası hazırda baxış prosesindədir.

---

## 🧪 Fayl Strukturu

```
├── manifest.json        — Manifest V3 konfiqurasiyası
├── background.js         — Service worker: inject məntiqi, badge, mesajlaşma
├── popup.html / popup.js — Toolbar popup UI-i
├── sayn-core.js          — Qlobal state, storage key-lər, ayarlar, utils
├── sayn-data.js          — Analiz mühərriki (NFB, mutual, diff, sort, export)
├── sayn-parser.js        — Instagram data eksport JSON parser
├── sayn-fetch.js         — Instagram private API qatı (Live Scan, unfollow)
├── sayn-ui.js            — Panel UI, naviqasiya, görünüşlər
├── sayn-styles.js        — Panel CSS-i (inject edilən stil modulu)
├── logo.png / SAYN.png   — İkon və brend aktivləri
└── index.html            — Bu landing page
```

---

## 📄 Lisenziya

Bütün hüquqlar qorunur © 2026 SAYN. NV Studio tərəfindən hazırlanıb.
