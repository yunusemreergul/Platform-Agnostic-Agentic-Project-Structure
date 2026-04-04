# Platform Agnostic Agentic Project Structure

*v1.1*

> Herhangi bir AI destekli geliştirme platformunda çalışabilen,
> taşınabilir ve modüler agentic proje yapısı şablonu.

---

## 1. Nedir ve Ne Amaçla Kullanılır

Bu şablon, AI agent'larının bir proje üzerinde etkili biçimde çalışabilmesi için gereken bağlam, hafıza, kural ve görev yapısını standart bir dizin düzeniyle tanımlar.

Temel hedefler:

- **Platform bağımsızlık** — Antigravity, Claude Code, Cursor veya herhangi bir agentic araçla çalışır. Platforma özel hiçbir bağımlılık içermez.
- **Taşınabilirlik** — `.context/` dizini olduğu gibi başka bir projeye kopyalanabilir.
- **Modülerlik** — Her dizin bağımsız çalışır. İhtiyaç duyulmayan modüller kullanılmadan bırakılabilir; geri kalan yapı bozulmaz.
- **Tek ve çok agent desteği** — Tek agent kullananlar için sade, çok agent kullananlar için koordinasyonlu bir yapı sunar.

---

## 2. Yapının Genel Felsefesi

### Giriş kapısı ve bağlam ayrımı

`AGENTS.md` projenin giriş kapısıdır ve proje kökünde durur. Platformların büyük çoğunluğu ilk olarak bu dosyayı arar. Dosyanın tek görevi projeyi tanıtmak ve `.context/` yapısına yönlendirmektir.

Tüm agentic bağlam `.context/` dizini altında toplanır. Bu ayrım sayesinde `.context/` kendi kendine yeten, bütünlüklü bir birim olur.

### Hafıza dosya sistemi üzerinde yaşar

LLM'lerin context window'u oturum bitince sıfırlanır. Kalıcı hafıza dosya sistemine yazılarak korunur. Agent her yeni oturumda ilgili hafıza dosyalarını okuyarak bağlamı yeniden kurar.

### Tetik her zaman kullanıcıdan gelir

`AGENTS.md` ile `.context/` altındaki tüm dosyalar — `rules/`, `agents/`, `memory/`, `tasks/` ve `skills/` — agent'ın bağlamını ve kısıtlarını tanımlar ancak otomatik aksiyon başlatmaz. Agent'ı harekete geçirmek için kullanıcı prompt vermek zorundadır. Bu dosyalar ne kadar iyi yazılırsa kullanıcının o kadar kısa ve net prompt vermesi yeterli olur. Örneğin agent tanım dosyası, kural dosyaları ve hafıza dosyaları doğru kurgulanmışsa "sıradaki görevi al" gibi çok kısa bir prompt yeterli olur — agent neyi, nerede, nasıl yapacağını zaten bağlamdan bilir.

### Zorunlu ve opsiyonel katmanlar

Tüm yapı iki katmandan oluşur:

- **Zorunlu çekirdek** — `AGENTS.md`, `memory/project-state.md`, `memory/progress.md`, `memory/decisions.md`
- **Opsiyonel modüller** — `rules/`, `agents/`, `memory/sessions/`, `tasks/`, `skills/`

Opsiyonel modüller ihtiyaca göre eklenir veya çıkarılır.

---

## 3. Dizin ve Dosya Şeması

### Şablon

```
project-root/
├── AGENTS.md
└── .context/
    ├── rules/
    │   └── [rule_name].md
    ├── agents/
    │   └── [agent_name].md
    ├── memory/
    │   ├── project-state.md
    │   ├── progress.md
    │   ├── decisions.md
    │   └── sessions/
    │       └── [agent_name]_session.md
    ├── tasks/
    │   ├── active/
    │   │   └── [task_name].md
    │   ├── backlog/
    │   │   └── [task_name].md
    │   └── completed/
    │       └── [task_name].md
    └── skills/
        └── [skill_name]/
            └── skill.md
```

### Dizin ve Dosya Açıklamaları

**`AGENTS.md`** — Zorunlu giriş kapısıdır. Projeyi, agent'ları ve `.context/` yapısını tanıtır. Her platform ilk olarak bu dosyayı arar.

**`.context/`** — Tüm agentic bağlam bu dizin altında toplanır. Platform bağımsız, taşınabilir birimdir.

**`rules/`** — Agent davranış kuralları. Her konu için ayrı bir dosya tutulur. Agent yalnızca ilgili kuralı okur, gereksiz bağlam yüklenmez.

**`rules/[rule_name].md`** — Tek bir konuya ait kural dosyası. Örnek: `code-style.md`, `security.md`, `workflow.md`, `testing.md`

**`agents/`** — Agent tanım dosyaları. Her agent'ın kimliği, kapsamı, sorumlulukları ve kısıtlamaları burada tanımlanır.

**`agents/[agent_name].md`** — Bir agent'a ait kimlik ve kapsam belgesi.

**`memory/`** — Proje geneli kalıcı hafıza. Oturumlar arasında bağlamın korunduğu yerdir.

**`memory/project-state.md`** — Projenin anlık genel durumu. Tüm agent'lar okur ve günceller. Zorunlu çekirdek dosyadır.

**`memory/progress.md`** — Görev durumu takibi. Tamamlanan, devam eden ve bekleyen işler. Zorunlu çekirdek dosyadır.

**`memory/decisions.md`** — Mimari kararlar ve gerekçeleri. Neden yapıldığını ileride anlamak için tutulur. Zorunlu çekirdek dosyadır.

**`memory/sessions/`** — Agent bazlı oturum hafızası. Her agent kendi oturumunu burada saklar.

**`memory/sessions/[agent_name]_session.md`** — Bir agent'ın oturum bazlı hafıza dosyası. Agent kapandığında durumunu buraya yazar, açıldığında buradan okur.

**`tasks/`** — Feature bazlı görev takibi. Her görev kendi dosyasında, durumuna göre ilgili alt dizinde bulunur. Görevlerin nasıl taşınacağı ve `progress.md` ile nasıl senkronize edileceği `AGENTS.md — Görev Akışı` bölümünde tanımlanır.

**`tasks/active/[task_name].md`** — Şu an üzerinde çalışılan görev.

**`tasks/backlog/[task_name].md`** — Planlanmış ama henüz başlanmamış görev.

**`tasks/completed/[task_name].md`** — Tamamlanan görev. Referans ve geçmiş karar alma için saklanır.

**`skills/`** — Agent yetenekleri. Her skill kendi dizininde, bağımsız ve genişlemeye açık şekilde tutulur.

**`skills/[skill_name]/skill.md`** — Skill tanımı, kapsamı ve kullanım rehberi.

### Örnek

```
project-root/
├── AGENTS.md
└── .context/
    ├── rules/
    │   ├── code-style.md
    │   ├── security.md
    │   ├── workflow.md
    │   └── testing.md
    ├── agents/
    │   ├── backend.md
    │   └── frontend.md
    ├── memory/
    │   ├── project-state.md
    │   ├── progress.md
    │   ├── decisions.md
    │   └── sessions/
    │       ├── backend_session.md
    │       └── frontend_session.md
    ├── tasks/
    │   ├── active/
    │   │   ├── user-auth.md
    │   │   └── product-listing.md
    │   ├── backlog/
    │   │   ├── payment-integration.md
    │   │   └── email-notifications.md
    │   └── completed/
    │       └── project-setup.md
    └── skills/
        ├── api-design/
        │   └── skill.md
        └── database/
            └── skill.md
```

---

## 4. Her Dosyanın İçerik Yapısı

### AGENTS.md

Projenin giriş kapısıdır. Kısa ve yönlendirici olmalıdır. Ayrıntılar `.context/` içindeki ilgili dosyalara bırakılır.

> **Tek agent kullananlar için not:** `Agent'lar` tablosu opsiyoneldir. Projede yalnızca bir agent varsa bu bölümü atlayabilir, `Oturum Başlangıcı`'nı da buna göre sadeleştirebilirsin.

```markdown
# [Proje Adı]

## Proje Hakkında
[Projenin ne yaptığını 2-3 cümleyle anlat.]

## Teknoloji Yığını
- [Dil / Framework]
- [Veritabanı]
- [Diğer temel bağımlılıklar]

## Agent'lar
<!-- Tek agent kullanıyorsan bu tabloyu silebilirsin -->
| Agent | Sorumluluk | Tanım Dosyası |
|---|---|---|
| [agent_name] | [Ne yapar] | .context/agents/[agent_name].md |

## Yapı Rehberi
- Davranış kuralları: `.context/rules/`
- Kalıcı hafıza: `.context/memory/`
- Görev takibi: `.context/tasks/`
- Yetenekler: `.context/skills/`

## Görev Akışı
Görev dosyaları, durumlarına göre üç farklı dizinde yaşar. Bir görevin durumu değiştiğinde **dosya taşıma ve `progress.md` güncellemesi tek bir adım (atomik işlem) olarak birlikte yapılmalıdır**:

1. **Backlog (Planlama):** Yeni görevler `tasks/backlog/` dizininde oluşturulur.
2. **Active (Geliştirme):** Agent görevi devraldığında dosyayı `tasks/active/` dizinine taşır ve `progress.md`'yi "Devam Eden" olarak günceller.
3. **Completed (Kapanış):** Görev bittiğinde dosya `tasks/completed/` dizinine taşınır, `progress.md` "Tamamlanan" olarak güncellenir ve görev `project-state.md`'deki "Aktif Çalışmalar" listesinden silinir.

## Oturum Başlangıcı
Her oturumda sırasıyla şunları oku:
1. `.context/memory/project-state.md`
2. `.context/memory/progress.md`
3. `.context/agents/[kendi_agent_dosyan].md` *(tek agent: bu adımı atlayabilirsin)*
4. `.context/memory/sessions/[kendi_agent_adın]_session.md` *(tek agent: opsiyonel)*
```

**Örnek — Task Manager projesi:**

```markdown
# Task Manager

## Proje Hakkında
Kullanıcıların proje ve görev oluşturup yönetebileceği,
ekip üyeleriyle işbirliği yapabileceği bir görev yönetim
uygulaması. Backend .NET 8 REST API, frontend React +
TypeScript ile geliştirilmektedir.

## Teknoloji Yığını
- **Backend:** .NET 8, Entity Framework Core, PostgreSQL
- **Frontend:** React, TypeScript
- **Auth:** JWT
- **Test:** xUnit (backend), Jest (frontend)

## Agent'lar
| Agent | Sorumluluk | Tanım Dosyası |
|---|---|---|
| backend | .NET API, veritabanı, auth | .context/agents/backend.md |
| frontend | React bileşenleri, sayfa yapısı, API entegrasyonu | .context/agents/frontend.md |

## Yapı Rehberi
- Davranış kuralları: `.context/rules/`
- Kalıcı hafıza: `.context/memory/`
- Görev takibi: `.context/tasks/`
- Yetenekler: `.context/skills/`

## Görev Akışı
Görev dosyaları, durumlarına göre üç farklı dizinde yaşar. Bir görevin durumu değiştiğinde **dosya taşıma ve `progress.md` güncellemesi tek bir adım (atomik işlem) olarak birlikte yapılmalıdır**:

1. **Backlog (Planlama):** Yeni görevler `tasks/backlog/` dizininde oluşturulur.
2. **Active (Geliştirme):** Agent görevi devraldığında dosyayı `tasks/active/` dizinine taşır ve `progress.md`'yi "Devam Eden" olarak günceller.
3. **Completed (Kapanış):** Görev bittiğinde dosya `tasks/completed/` dizinine taşınır, `progress.md` "Tamamlanan" olarak güncellenir ve görev `project-state.md`'deki "Aktif Çalışmalar" listesinden silinir.

## Oturum Başlangıcı
Her oturumda sırasıyla şunları oku:
1. `.context/memory/project-state.md`
2. `.context/memory/progress.md`
3. `.context/agents/[kendi_agent_dosyan].md`
4. `.context/memory/sessions/[kendi_agent_adın]_session.md`
```

---

### .context/rules/[rule_name].md

Her kural dosyası tek bir konuya odaklanır. Kurallar kesin ve ölçülebilir olmalıdır. İsteğe bağlı olarak tek dosyada (`code-style.md`) veya konu bazlı ayrı dosyalarda (`backend-code-style.md`, `frontend-code-style.md`) tutulabilir.

```markdown
# [Kural Başlığı]

## Kapsam
[Bu kuralın geçerli olduğu alan]

## Zorunlular
- [Net, ölçülebilir kural]
- [Net, ölçülebilir kural]

## Yasaklar
- [Kesinlikle yapılmayacak şey]
- [Kesinlikle yapılmayacak şey]

## İstisnalar
- [Varsa istisnalar]
```

Örnek dosya isimleri: `code-style.md`, `security.md`, `workflow.md`, `testing.md`

**Örnek — code-style.md:**

```markdown
# Code Style

## Kapsam
Backend (.NET 8) ve frontend (React + TypeScript) genelinde kod yazım standartları.

## Zorunlular
- Tüm kod yorumları İngilizce yazılır
- Kullanıcıya yönelik metinler Türkçe yazılır
- Max satır uzunluğu 120 karakter
- C# naming: PascalCase sınıf/metod, camelCase değişken
- Her public metod XML doc comment içermeli
- async/await pattern zorunlu
- LINQ sorguları method syntax ile yazılır
- React bileşenleri functional component olarak yazılır
- Props için interface tanımı zorunlu
- Dosya isimleri: bileşenler PascalCase, yardımcı fonksiyonlar camelCase

## Yasaklar
- .Result veya .Wait() kullanılmaz
- Magic number kullanılmaz, sabitler const veya enum ile tanımlanır
- any tipi kullanılmaz, strict TypeScript zorunlu
- Inline style kullanılmaz, CSS Modules veya Tailwind kullanılır

## İstisnalar
- Test dosyalarında XML doc comment zorunlu değil
```

**Örnek — security.md:**

```markdown
# Security

## Kapsam
Backend ve frontend genelinde güvenlik kuralları.

## Zorunlular
- Secret ve credential'lar environment variable ile yönetilir
- Tüm endpoint'ler varsayılan olarak [Authorize] ile korunur
- Public endpoint'ler açıkça [AllowAnonymous] ile işaretlenir
- SQL sorguları parametreli yazılır
- Kullanıcı girdisi FluentValidation ile doğrulanır
- JWT access token süresi max 1 saat, refresh token max 7 gün
- Kullanıcı verisi sessionStorage'a kaydedilir

## Yasaklar
- .env dosyaları git'e eklenmez
- Raw SQL kullanılmaz
- API anahtarları frontend kodunda bulunmaz
- localStorage'a kullanıcı verisi yazılmaz
- dangerouslySetInnerHTML kullanılmaz

## İstisnalar
- Dependency güvenlik açıkları haftalık kontrol edilir, acil durumlarda önceliklendirilir
```

**Örnek — workflow.md:**

```markdown
# Workflow

## Kapsam
Geliştirme süreci, git kullanımı ve hafıza yönetimi kuralları.

## Zorunlular
- Commit mesajları Conventional Commits formatında yazılır: feat/fix/chore/docs/refactor
- Her commit tek bir sorumluluğu kapsar
- Branch isimleri: feature/[task-adı] veya fix/[task-adı]
- Yeni görev alındığında progress.md güncellenir
- Mimari karar alındığında decisions.md güncellenir
- Oturum sonunda sessions/[agent_adı]_session.md güncellenir

## Yasaklar
- git push doğrudan çalıştırılmaz, kullanıcıya önerilir
- Var olan kod silinmeden önce kullanıcıya sorulmaz
- Tek seferde 200 satırdan fazla kod üretilmez
- Belirsiz gereksinim olduğunda sormadan yazmaya başlanmaz
- appsettings.Production.json dosyasına dokunulmaz

## İstisnalar
- Büyük refactor öncesi değişikliğin kapsamı kullanıcıya açıklanır, onay alınır
```

**Örnek — testing.md:**

```markdown
# Testing

## Kapsam
Backend (xUnit) ve frontend (Jest) test yazım standartları.

## Zorunlular
- Her yeni public fonksiyon için unit test yazılır
- Test isimleri [Metod]_[Senaryo]_[BeklenenSonuç] formatında olur
- Test verisi sabit değerlerle tanımlanır
- Her servis sınıfı için ayrı test sınıfı oluşturulur
- Integration test'ler ayrı dizinde tutulur: tests/Integration/
- Bileşen testleri React Testing Library ile yazılır
- API çağrıları mock'lanır

## Yasaklar
- Repository katmanı test edilirken gerçek DB'ye bağlanılmaz, mock kullanılır
- Random test verisi kullanılmaz
- Snapshot test kullanılmaz
- Coverage %80'in altına düşürülmez

## İstisnalar
- Kullanıcı etkileşimleri test edilir, implementasyon detayı test edilmez
```

---

### .context/agents/[agent_name].md

Her agent'ın kimlik ve kapsam belgesidir. Agent bu dosyayı okuyarak ne yapacağını ve neye dokunamayacağını anlar.

```markdown
# [Agent Adı]

## Kimlik
- **Sorumluluk:** [Bu agent ne yapar]
- **Kapsam:** [Hangi dizin ve dosyalardan sorumlu]

## Sahip Olduğu Dizinler (yazma yetkisi)
- `[dizin_1]/`
- `[dizin_2]/`

## Dokunmayacağı Dizinler
- `[dizin]/` — [neden]

## Kullandığı Skills
- `.context/skills/[skill_name]/skill.md`

## Oturum Başlangıcı
1. `.context/memory/project-state.md` oku
2. `.context/memory/progress.md` oku
3. `.context/memory/sessions/[agent_name]_session.md` oku

## Hafıza Yazma Kuralları
- Yeni görev alındığında → `progress.md` güncelle
- Mimari karar alındığında → `decisions.md` güncelle
- Oturum sonunda → `sessions/[agent_name]_session.md` güncelle
```

**Örnek — backend.md:**

```markdown
# Backend Agent

## Kimlik
- **Sorumluluk:** .NET 8 REST API geliştirme, veritabanı yönetimi, kimlik doğrulama
- **Kapsam:** API endpoint'leri, servis katmanı, repository katmanı, veritabanı migration'ları

## Sahip Olduğu Dizinler (yazma yetkisi)
- `src/backend/`
- `tests/backend/`
- `.context/memory/sessions/backend_session.md`

## Dokunmayacağı Dizinler
- `src/frontend/` — frontend agent sorumlu
- `tests/frontend/` — frontend agent sorumlu
- `infra/` — manuel yönetim
- `appsettings.Production.json` — manuel yönetim

## Kullandığı Skills
- `.context/skills/api-design/skill.md`
- `.context/skills/database/skill.md`

## Oturum Başlangıcı
1. `.context/memory/project-state.md` oku
2. `.context/memory/progress.md` oku
3. `.context/memory/sessions/backend_session.md` oku

## Hafıza Yazma Kuralları
- Yeni görev alındığında → `progress.md` güncelle
- Mimari karar alındığında → `decisions.md` güncelle
- Oturum sonunda → `sessions/backend_session.md` güncelle
```

**Örnek — frontend.md:**

```markdown
# Frontend Agent

## Kimlik
- **Sorumluluk:** React + TypeScript ile kullanıcı arayüzü geliştirme, API entegrasyonu
- **Kapsam:** React bileşenleri, sayfa yapısı, state yönetimi, API istemcisi

## Sahip Olduğu Dizinler (yazma yetkisi)
- `src/frontend/`
- `tests/frontend/`
- `.context/memory/sessions/frontend_session.md`

## Dokunmayacağı Dizinler
- `src/backend/` — backend agent sorumlu
- `tests/backend/` — backend agent sorumlu
- `infra/` — manuel yönetim

## Kullandığı Skills
- `.context/skills/ui-components/skill.md`

## Oturum Başlangıcı
1. `.context/memory/project-state.md` oku
2. `.context/memory/progress.md` oku
3. `.context/memory/sessions/frontend_session.md` oku

## Hafıza Yazma Kuralları
- Yeni görev alındığında → `progress.md` güncelle
- Mimari karar alındığında → `decisions.md` güncelle
- Oturum sonunda → `sessions/frontend_session.md` güncelle
```

---

### .context/memory/project-state.md

Projenin anlık genel durumunu, aktif çalışmaları, bilinen kısıtları ve ajanlar arası iletişim için **İş Devirlerini (Handoff)** tutan paylaşılan durum dosyasıdır. Ajanlar birbirlerinden bağımsız çalışsalar da, bir ajan işi belirli bir aşamaya getirdiğinde diğer ajana paslaması gereken teknik detayları (API hazır, tasarım bitti vb.) "Bekleyen Handoff'lar" tablosuna yazar ve böylece diğer ajan işe başladığında tam olarak kaldığı noktayı bulur. Tüm agent'lar okur; görev durumu veya proje genelinde bir şey değiştiğinde ilgili agent günceller. Her güncellemede `## Son Güncelleme` bölümü önce yenilenir.

```markdown
# Project State

## Son Güncelleme
- **Agent:** [agent_name]
- **Tarih:** YYYY-MM-DD HH:MM
- **Değişiklik:** [Ne yapıldı, bir cümle]

## Genel Durum
[Projenin şu anki durumu: aktif geliştirme / stabilizasyon / bakım]

## Aktif Çalışmalar
| Agent | Dizin / Dosya | Durum |
|---|---|---|
| [agent_name] | `[dizin/]` | Devam ediyor |

## Bekleyen Handoff'lar (İş Devirleri)
| Kime (Hedef) | Kimden (Kaynak) | Konu / Mesaj | İlgili Dosya veya Task |
|---|---|---|---|
| [hedef_agent] | [kaynak_agent] | [Kısa bilgi ve devredilen işin detayı] | `[dosya_veya_task_yolu]` |

## Bilinen Kısıtlar
- [Dikkat edilmesi gereken şey]
```

**Örnek — project-state.md:**

```markdown
# Project State

## Son Güncelleme
- **Agent:** backend
- **Tarih:** 2025-01-20 10:30
- **Değişiklik:** Auth endpoint'leri tamamlandı, JWT implementasyonu yapıldı

## Genel Durum
Aktif geliştirme — auth katmanı tamamlandı, görev yönetimi geliştirmesi devam ediyor.

## Aktif Çalışmalar
| Agent | Dizin / Dosya | Durum |
|---|---|---|
| backend | `src/backend/Features/Tasks/` | Devam ediyor |
| frontend | `src/frontend/pages/auth/` | Devam ediyor |

## Bekleyen Handoff'lar (İş Devirleri)
| Kime (Hedef) | Kimden (Kaynak) | Konu / Mesaj | İlgili Dosya veya Task |
|---|---|---|---|
| frontend | backend | Auth endpoint'leri `/api/auth/login` ve `/register` tamamlandı. JWT response yapısı eklendi, entegrasyonuna başlanabilir. | `tasks/active/auth-pages.md` |

## Bilinen Kısıtlar
- PostgreSQL bağlantısı sadece local ortamda yapılandırıldı, staging henüz yok
- Email bildirimleri v2'ye ertelendi
- `appsettings.Production.json` manuel yönetimde, agent dokunmaz
```

---

### .context/memory/progress.md

Görevlerin genel durumunu takip eder. `tasks/` dizinindeki detaylı görev dosyalarına referans verir. Her güncellemede `## Son Güncelleme` bölümü önce yenilenir.

```markdown
# Progress

## Son Güncelleme
- **Agent:** [agent_name]
- **Tarih:** YYYY-MM-DD HH:MM
- **Değişiklik:** [Ne yapıldı, bir cümle]

## Tamamlanan
- [x] [Görev adı] — YYYY-MM-DD

## Devam Eden
- [ ] [Görev adı] — `tasks/active/[task_name].md`
  **Agent:** [agent_name]

## Bekleyen
- [ ] [Görev adı] — `tasks/backlog/[task_name].md`
  **Agent:** [agent_name]

## İptal / Ertelenen
- [ ] ~~[Görev adı]~~ — [Gerekçe] — YYYY-MM-DD
```

**Örnek — progress.md:**

```markdown
# Progress

## Son Güncelleme
- **Agent:** backend
- **Tarih:** 2025-01-20 10:30
- **Değişiklik:** JWT auth tamamlandı, görev CRUD endpoint'leri devam ediyor

## Tamamlanan
- [x] Proje iskeleti oluşturuldu — 2025-01-15
- [x] Veritabanı şeması tasarlandı — 2025-01-16
- [x] Entity Framework migration'ları oluşturuldu — 2025-01-17
- [x] JWT auth implementasyonu — 2025-01-20

## Devam Eden
- [ ] Görev CRUD endpoint'leri — `tasks/active/task-crud-api.md`
  **Agent:** backend
- [ ] Login ve kayıt sayfaları — `tasks/active/auth-pages.md`
  **Agent:** frontend

## Bekleyen
- [ ] Proje yönetimi endpoint'leri — `tasks/backlog/project-management-api.md`
  **Agent:** backend
- [ ] Görev listesi sayfası — `tasks/backlog/task-list-page.md`
  **Agent:** frontend
- [ ] Kullanıcı profil sayfası — `tasks/backlog/user-profile-page.md`
  **Agent:** frontend

## İptal / Ertelenen
- [ ] ~~Email bildirimleri~~ — v2'ye ertelendi — 2025-01-18
```

---

### .context/memory/decisions.md

Mimari kararların ve gerekçelerinin saklandığı dosyadır. Neden bu kararın alındığı ileride sorgulandığında bu dosya başvuru kaynağıdır. Her yeni karar eklenirken `## Son Güncelleme` bölümü önce yenilenir.

```markdown
# Decisions

## Son Güncelleme
- **Agent:** [agent_name]
- **Tarih:** YYYY-MM-DD HH:MM
- **Değişiklik:** [Ne yapıldı, bir cümle]

### YYYY-MM-DD — [Başlık]
**Karar:** [Ne yapılacak]
**Gerekçe:** [Neden]
**Alternatifler:** [Değerlendirilen ama seçilmeyen seçenekler]
**Etkilenen dosyalar:** [Hangi modüller]
```

**Örnek — decisions.md:**

```markdown
# Decisions

## Son Güncelleme
- **Agent:** backend
- **Tarih:** 2025-01-20 10:30
- **Değişiklik:** JWT yapılandırma kararı eklendi

### 2025-01-16 — Veritabanı Şeması
**Karar:** Soft delete pattern kullanılacak, kayıtlar fiziksel olarak silinmeyecek
**Gerekçe:** Görev geçmişi ve audit trail ihtiyacı, silinen verilerin raporlarda görünmemesi gerekiyor
**Alternatifler:** Hard delete (audit trail kaybı riski), ayrı archive tablosu (karmaşıklık)
**Etkilenen dosyalar:** `src/backend/Data/Entities/`, tüm repository sınıfları

### 2025-01-17 — Repository Pattern
**Karar:** Generic repository pattern uygulanacak
**Gerekçe:** Kod tekrarını azaltmak, test edilebilirliği artırmak
**Alternatifler:** DbContext doğrudan kullanım (test zorluğu), CQRS (overkill bu aşamada)
**Etkilenen dosyalar:** `src/backend/Data/Repositories/`

### 2025-01-20 — JWT Yapılandırması
**Karar:** Access token 1 saat, refresh token 7 gün
**Gerekçe:** Güvenlik ve kullanıcı deneyimi dengesi
**Alternatifler:** 15 dk access token (çok kısa, UX kötü), 24 saat (güvenlik riski)
**Etkilenen dosyalar:** `src/backend/Features/Auth/`
```

---

### .context/memory/sessions/[agent_name]_session.md

Her agent'ın oturum bazlı hafızasıdır. Agent kapandığında durumunu buraya yazar, açıldığında buradan okuyarak kaldığı yerden devam eder.

```markdown
# [Agent Adı] — Oturum Hafızası

## Son Oturum
- **Tarih:** YYYY-MM-DD
- **Yapılan:** [Kısa özet]

## Kaldığım Yer
- **Dosya:** `[dosya_yolu]`
- **Fonksiyon / Bölüm:** [Nerede kaldı]
- **Sonraki Adım:** [Ne yapılacak]
- **Bağımlılık:** [Varsa önce tamamlanması gereken şey]

## Öğrenilen / Dikkat Edilecek
- [Bir sonraki oturumda hatırlanması gereken şey]
```

**Örnek — backend_session.md:**

```markdown
# Backend Agent — Oturum Hafızası

## Son Oturum
- **Tarih:** 2025-01-20
- **Yapılan:** JWT auth implementasyonu tamamlandı, login ve register endpoint'leri yazıldı, token yenileme mekanizması eklendi

## Kaldığım Yer
- **Dosya:** `src/backend/Features/Tasks/TasksController.cs`
- **Fonksiyon / Bölüm:** GET /api/tasks endpoint'i yarım kaldı, filtreleme parametreleri eklenmedi
- **Sonraki Adım:** TasksController'a status ve assignee filtreleme ekle, ardından POST /api/tasks yaz
- **Bağımlılık:** TaskService.GetFilteredTasksAsync metodu önce tamamlanmalı

## Öğrenilen / Dikkat Edilecek
- EF Core'da soft delete için global query filter eklendi — yeni entity eklerken bu filter'ı unutma
- PostgreSQL bağlantı string'i appsettings.Development.json'da, Production'a dokunma
```

**Örnek — frontend_session.md:**

```markdown
# Frontend Agent — Oturum Hafızası

## Son Oturum
- **Tarih:** 2025-01-20
- **Yapılan:** Proje yapısı oluşturuldu, React Router kuruldu, axios ile API istemcisi yapılandırıldı

## Kaldığım Yer
- **Dosya:** `src/frontend/pages/auth/LoginPage.tsx`
- **Fonksiyon / Bölüm:** Form validasyonu yarım kaldı, hata mesajları gösterilmiyor
- **Sonraki Adım:** React Hook Form ile validasyon tamamla, ardından RegisterPage.tsx yaz
- **Bağımlılık:** Backend auth endpoint'leri hazır, API istemcisi yapılandırıldı

## Öğrenilen / Dikkat Edilecek
- JWT token sessionStorage'a kaydediliyor, localStorage kullanılmaz (security.md kuralı)
- API base URL .env dosyasından okunuyor: VITE_API_URL
```

---

### .context/tasks/[active|backlog|completed]/[task_name].md

Her görev kendi dosyasında yaşar ve durumuna göre ilgili dizinde bulunur. Görevle ilgili bir tıkanıklık (blocker) yaşanırsa, bu durum dosyanın içindeki `## Blocker` bölümünde belgelenir, böylece diğer agent'lar veya kullanıcı sorunu anında görebilir.

```markdown
# [Görev Adı]

## Tanım
[Bu görev ne yapmayı amaçlıyor, 2-3 cümle]

## Agent
[Bu görevi hangi agent üstleniyor]

## Kabul Kriterleri
- [ ] [Görevin tamamlanmış sayılması için gereken şart]
- [ ] [Şart]

## Adımlar
- [ ] [Adım 1]
- [ ] [Adım 2]

## Bağımlılıklar
- [Bu görev başlamadan önce tamamlanması gereken şey]

## Blocker
- **Durum:** Yok *(veya: Bekliyor)*
- **Neden:** — *(varsa: ne tıkadı)*

## Notlar
- [Geliştirme sürecinde ortaya çıkan önemli bilgiler]
```

**Örnek — tasks/active/task-crud-api.md:**

```markdown
# Görev CRUD API

## Tanım
Kullanıcıların görev oluşturabilmesi, güncelleyebilmesi, listeleyebilmesi ve silebilmesi için gerekli REST API endpoint'lerinin geliştirilmesi. Soft delete pattern uygulanacak.

## Agent
backend

## Kabul Kriterleri
- [x] GET /api/tasks — filtreleme (status, assignee) ve sayfalama destekli
- [x] GET /api/tasks/{id} — tek görev detayı
- [ ] POST /api/tasks — yeni görev oluşturma
- [ ] PUT /api/tasks/{id} — görev güncelleme
- [ ] DELETE /api/tasks/{id} — soft delete

## Adımlar
- [x] TaskService.GetFilteredTasksAsync metodunu tamamla
- [x] TasksController GET endpoint'lerine filtreleme ekle
- [ ] POST /api/tasks endpoint'ini yaz
- [ ] PUT /api/tasks/{id} endpoint'ini yaz
- [ ] DELETE /api/tasks/{id} soft delete uygula
- [ ] xUnit testleri yaz

## Bağımlılıklar
- JWT auth tamamlandı, [Authorize] attribute kullanılabilir
- Generic repository hazır

## Blocker
- **Durum:** Yok
- **Neden:** —

## Notlar
- Soft delete için IsDeleted global query filter zaten var, yeni sorgu yazmaya gerek yok
```

**Örnek — tasks/backlog/project-management-api.md:**

```markdown
# Proje Yönetimi API

## Tanım
Kullanıcıların proje oluşturup yönetebilmesi, proje üyelerini ekleyip çıkarabilmesi için REST API endpoint'lerinin geliştirilmesi.

## Agent
backend

## Kabul Kriterleri
- [ ] GET /api/projects — kullanıcının projelerini listele
- [ ] POST /api/projects — yeni proje oluştur
- [ ] PUT /api/projects/{id} — proje güncelle
- [ ] DELETE /api/projects/{id} — soft delete
- [ ] POST /api/projects/{id}/members — üye ekle
- [ ] DELETE /api/projects/{id}/members/{userId} — üye çıkar

## Adımlar
- [ ] Project entity ve migration oluştur
- [ ] ProjectRepository yaz
- [ ] ProjectService yaz
- [ ] ProjectsController yaz
- [ ] xUnit testleri yaz

## Bağımlılıklar
- Görev CRUD API tamamlanmalı (task-crud-api.md)

## Blocker
- **Durum:** Bekliyor
- **Neden:** Görev CRUD API henüz tamamlanmadı

## Notlar
- Proje sahibi otomatik olarak ilk üye olarak eklenmeli
```

**Örnek — tasks/completed/project-setup.md:**

```markdown
# Proje İskeleti

## Tanım
Backend ve frontend proje yapılarının oluşturulması, temel bağımlılıkların kurulması ve geliştirme ortamının yapılandırılması.

## Agent
backend, frontend

## Kabul Kriterleri
- [x] .NET 8 Web API projesi oluşturuldu
- [x] React + TypeScript projesi oluşturuldu
- [x] PostgreSQL bağlantısı yapılandırıldı
- [x] Entity Framework Core kuruldu
- [x] Temel proje dizin yapısı oluşturuldu
- [x] .gitignore yapılandırıldı

## Tamamlanma
- **Tarih:** 2025-01-15
- **Notlar:** Monorepo yapısı benimsendi, src/backend ve src/frontend altında ayrı projeler
```

---

### .context/skills/[skill_name]/skill.md

Her skill agent'ın belirli bir alanda nasıl davranacağını tanımlar.

```markdown
# [Skill Adı]

## Kapsam
[Bu skill hangi tür görevleri kapsar]

## Yaklaşım
[Bu alanda nasıl düşünülmeli, hangi prensipler izlenmeli]

## Adımlar
1. [Adım]
2. [Adım]

## Dikkat Edilecekler
- [Bu alanda yapılan yaygın hatalar]

## Referanslar
- [Varsa ilgili döküman veya kaynak]
```

**Örnek — skills/api-design/skill.md:**

```markdown
# API Design

## Kapsam
REST API endpoint tasarımı, request/response şemaları, hata yönetimi ve API versiyonlama.

## Yaklaşım
- RESTful prensiplere uy: doğru HTTP metodları, anlamlı URL yapısı
- Resource odaklı tasarım: /api/tasks, /api/projects gibi isimler kullan
- Tutarlı response formatı her endpoint'te aynı yapıda olmalı
- Hata mesajları kullanıcı dostu ve hata kodu içermeli

## Adımlar
1. Resource'u tanımla ve URL yapısını belirle
2. HTTP metodlarını ata (GET, POST, PUT, DELETE)
3. Request body ve query parametrelerini tanımla
4. Response şemasını ve HTTP status kodlarını belirle
5. Hata senaryolarını ve response'larını tanımla
6. [Authorize] gerekliliğini değerlendir

## Dikkat Edilecekler
- URL'lerde fiil kullanılmaz: /api/getTasks değil /api/tasks
- Sayfalama için page ve pageSize query parametreleri kullanılır
- Silme işlemleri soft delete ile yapılır, hard delete yapılmaz
- Response body her zaman tutarlı bir wrapper içinde döner

## Referanslar
- `.context/rules/security.md` — endpoint güvenlik kuralları
- `.context/rules/testing.md` — API test kuralları
```

**Örnek — skills/database/skill.md:**

```markdown
# Database

## Kapsam
Entity Framework Core ile veritabanı tasarımı, migration yönetimi ve sorgu optimizasyonu.

## Yaklaşım
- Code-first yaklaşım: entity'ler önce yazılır, migration'lar oluşturulur
- Her entity için ayrı configuration sınıfı (IEntityTypeConfiguration)
- Soft delete pattern: IsDeleted ve DeletedAt alanları her entity'de bulunur
- Global query filter ile silinen kayıtlar otomatik filtrelenir

## Adımlar
1. Entity sınıfını oluştur (src/backend/Data/Entities/)
2. IEntityTypeConfiguration implementasyonunu yaz
3. DbContext'e DbSet ekle
4. Migration oluştur: dotnet ef migrations add [MigrationName]
5. Migration'ı uygula: dotnet ef database update
6. Repository metodlarını yaz

## Dikkat Edilecekler
- Migration isimleri açıklayıcı olmalı: AddTaskTable, AddProjectMembersTable
- N+1 sorgu probleminden kaçın, Include ile eager loading kullan
- Büyük veri setleri için AsNoTracking() kullan
- Soft delete global query filter tüm entity'lere uygulanmalı — yeni entity eklerken unutma

## Referanslar
- `.context/memory/decisions.md` — soft delete ve repository pattern kararları
- `.context/rules/testing.md` — repository test kuralları
```

---

## 5. Kullanım Rehberi

### Yeni projeye entegrasyon

1. `AGENTS.md` dosyasını `project-root/` altına oluştur
2. `.context/` dizinini proje köküne kopyala
3. `AGENTS.md` içindeki `[yer tutucu]` alanları doldur
4. İhtiyaç duyulmayan modülleri (`rules/`, `skills/` vb.) boş bırak veya sil
5. Zorunlu hafıza dosyalarını (`project-state.md`, `progress.md`, `decisions.md`) doldur

### Tek agent kullanımı

`agents/` dizininde tek bir dosya yeterlidir. `sessions/` dizini opsiyoneldir ancak uzun soluklu projelerde oturum sürekliliği için önerilir.

Minimal `AGENTS.md` yapısı (tek agent için):
- `Agent'lar` tablosu kaldırılabilir
- `Oturum Başlangıcı` listesi 2 adıma indirilebilir (`project-state.md` ve `progress.md`)
- `agents/` dizini de opsiyoneldir; kurallar doğrudan `rules/` ile yönetilebilir

### Multi-agent kullanımı

Her agent için `agents/[agent_name].md` ve `sessions/[agent_name]_session.md` dosyaları oluşturulur. `project-state.md` içindeki "Aktif Çalışmalar" tablosu agent'lar arası koordinasyonu sağlar. Her agent yalnızca kendi tanım dosyasında belirtilen dizinlere yazar.

### Görev akışı

Görev yaşam döngüsü ve `progress.md` senkronizasyon kuralları `AGENTS.md — Görev Akışı` bölümünde tanımlıdır. Bu bölüm her projede zorunlu olduğundan kural tek bir yerde yaşar ve her agent oturum başında okur.

Kısaca:
```
tasks/backlog/ → tasks/active/ → tasks/completed/
```
Her geçişte dosya taşıma + `progress.md` güncellemesi birlikte yapılır. Ayrıntı için `AGENTS.md`'ye bakınız.

### Agent'a verilecek ilk prompt

```
.context/agents/[agent_name].md dosyasını oku,
ardından progress.md'deki sıradaki görevi al ve
ne yapacağını önce bana söyle, onaylarsam başla.
```

---
