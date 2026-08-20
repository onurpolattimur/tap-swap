# RUH KOŞUCUSU — Tasarım Dokümanı

**Sürüm:** v3 — "Her Gece Aynı Yol" · **Tarih:** 2026-08-20 · **Durum:** Oynanabilir prototip (tek dosya)

> Bu doküman, oyunun dış bir gözle (insan ya da AI) değerlendirilmesi için hazırlandı.
> Amaç: tasarımın güçlü/zayıf yanları, kompulsiyon döngüsünün işleyip işlemediği ve
> gözden kaçan fırsatlar hakkında yorum almak. Dokümanın sonunda değerlendiriciye
> yönelik somut sorular var.

---

## 1. Kimlik ve Teknik Çerçeve

| | |
|---|---|
| Tür | Tek parmakla oynanan, tap-to-possess koşu oyunu (mobil öncelikli) |
| Platform | HTML5 canvas, tek dosya (`index.html`, ~1500 satır), harici bağımlılık yok |
| Görsel | El yapımı piksel sprite'lar, 240×160 mantıksal çözünürlük, tam ekrana ölçeklenir |
| Ses | WebAudio ile prosedürel chiptune SFX + basit müzik |
| Kayıt | localStorage (şema v3) |
| Seans hedefi | 30–90 saniyelik run'lar, "bir gece daha" döngüsü |
| Monetizasyon iskeleti | Rewarded ad simülasyonu (diriliş), kurgu içinde çerçeveli |

## 2. Çekirdek Fantezi

**Ölmüş bir ruhsun. Her gece, memleketten evine uzanan aynı uzun yolu yürüyorsun.**

- Ruh tek başına ilerleyemez; yaşayanların bedenlerini **ödünç alır** (dokun → geç).
  Eski beden koşmaya devam eder ve genelde ölür — fiil, bakımı değil **terk etmeyi** öğretir.
- Beden düşünce ölmezsin (zaten ölüsün): **gece biter**, yeni gecede yolun başında
  yeniden toplanırsın. "Bir run daha" döngüsü kurgunun kendisidir — musallat olmak
  (haunting) bir tekrar döngüsüdür.
- Eve **bütün oyunda bir kez** varılır. "EVDESİN" kelimesi tek bir kez söylenir.
  Varış = döngüden azat. Oyunun sonu vardır ve bu bilinçli bir karardır.

## 3. Saniye Saniye Oynanış

1. Bedenler otomatik sağa koşar (koşucu 34, bisiklet 60, kuş 55, araba 82, kamyon 46 birim/sn × biyom çarpanı).
2. Dört şerit: tel (kuş) / yol (araba+kamyon) / bisiklet yolu / kaldırım (koşucu). Her şeridin kendi engeli var (direk / bariyer / çukur / koni). Kamyon bariyerleri ezer.
3. Tek input: ekrana dokun → en yakın bedene ruh atlar (yarıçap 34px; ölmek üzere olan bedene yanlışlıkla geçmeyi zorlaştıran skorlama var).
4. Bedenler yorulur (stamina; tip başına 3.2–5.5 sn taban, biyom drain çarpanıyla). Stamina biterse beden düşer → gece biter.
5. Yoldaki **nefes ışıkları** anında stamina tazeler (+0.4; toplanabilir, sayaç yok). Işık kümelerinin %50'si bilerek engelli şeride dizilir: "güvenli şerit mi, nefesli şerit mi?"
6. **Adil oyun garantisi:** tehlike yakınken ekranda geçilebilir beden yoksa sistem bir tane doğurur.
7. Yol 8300 birim, 10 biyomdan geçer, biyom sınırlarında fenerler durur. HUD'da tek büyük sayı: **EVE [kalan]**.

## 4. Sistemler

### 4.1 Rota (tek kesintisiz yol)

| # | Biyom | Tema | Uzunluk | Hız çarpanı | Drain | Kamyon | Not |
|---|---|---|---|---|---|---|---|
| 1 | PARK | gündüz | 420 | 0.72 | 0 | 0 | öğretici bölge, rehber el |
| 2 | KORU | gündüz | 560 | 0.78 | 0.30 | 0 | drain başlar |
| 3 | YOL | gündüz | 700 | 0.84 | 0.45 | 0.12 | |
| 4 | ANAYOL | günbatımı | 800 | 0.90 | 0.55 | 0.12 | |
| 5 | KONVOY | günbatımı | 900 | 0.95 | 0.65 | **0.30** | kamyon bölgesi |
| 6 | FENER | günbatımı | 980 | 1.00 | 0.75 | 0.15 | dinç beden bolluğu (1.6×) |
| 7 | NEON | gece | 1060 | 1.06 | 0.85 | 0.15 | |
| 8 | AY | gece | 1140 | 1.12 | 0.95 | 0.20 | |
| 9 | SON YOL | gece | 1220 | 1.18 | 1.05 | 0.20 | dinç beden 1.3× |
| 10 | ŞAFAK | şafak | 520 | 0.85 | **0** | 0.10 | kutsal final: zincir ve altın misafir susar |

Biyom geçişi akış içinde olur (tema/parametre değişir, overlay yok). Ufukta ev silüeti her biyomda büyür; akşamdan sonra penceresi yanar ("gecede evi ışığından tanırsın").

### 4.2 Kompulsiyon katmanı

- **Rekor tablosu:** En uzak nokta (`bestRel`) kalıcı. Yolda o noktada bayrak + **oturan kedi** çizilir. Ölüm ekranı: "rekoruna X kalmıştı" / "YENİ REKOR +X". Rekoru canlı geçince: REKOR patlaması + kedi 3 saniye yanında koşar.
- **Sınır bölgesi:** Rekorun ötesinde nefes ışıkları daha sık (spawn aralığı 90+70 → 50+40) ve dinç beden şansı 1.5× — iyi şeyler yalnızca hiç gidilmemiş yerde.
- **Son Nefes zinciri (push-your-luck):** Terk edilen beden ≤0.6 sn içinde ölürse "mükemmel çıkış": beden son nefesini verir (+0.3 stamina), zincir büyür. Zincir ×3 → yolda bir AN belirir. 0.6–1.2 sn arası çıkışa cezasız "az kaldı" halkası. Erken/güvenli çıkış zinciri sıfırlar. Her tap bir bahistir.
- **Altın misafirler:** ~35–60 sn'de bir nadir altın beden belirir (aura + yumuşak çan), 7 sn'de kaybolur. Ona geçmek 6 sn lütuf: yorulmazlık + geniş nefes mıknatısı (yarıçap 26). Kaçırılırsa ölüm ekranı: "altın misafir kaçtı — belki yarın gece."
- **Sıfır sürtünme:** Ölüm ekranında canvas'a tek dokunuş = yeni gece.

### 4.3 Anlar (toplanabilir meta)

- Yol boyunca **30 sabit "büyük an" noktası** (biyom başına 3, riskli kesimlerde; parlayan yıldız). Toplayınca kısa bir zaman-durması; anlar ruhun ardında **altın kıvılcım kuyruğu** olarak süzülür (taşınan değer her karede ekranda).
- **Fenerden geçerken** taşınan anlar **eve gönderilir** (`momsHome`); eve giden an noktası bir daha doğmaz. Ev eşikleri: 3/7/12/17/22/27 an → çit → çiçekler → kedi → ışıklar → ağaç → çatı penceresi (her biri bir anı cümlesiyle: "Kediyi hatırladın — biri seni bekliyor.").
- **Ölünce** taşınan anlar düştüğün yerde bekler (kalıcı, kaybolmaz); sonraki gece üstünden geçince kavuşursun.
- **EMANET (harcama ikilemi):** 3+ an taşırken fenere yaklaşınca davet belirir; fenere dokunursan 3 anını **bırakırsın** → fener mühürlenir, geceler artık oradan başlar. Bedel gerçek: o anlar evi asla aydınlatmayacak. Dil bilinçli: "öde/satın al" yasak, "bırak/emanet" serbest. Ölüm ekranında "BAŞTAN YÜRÜ" seçeneği var (baştan yürüyene ilk 1200 birimde nefes cömertliği).
- Zincir ödülü anlar (idx −1) sonsuz kaynaktır ama ev eşiği 27'de doyar.

### 4.4 Yol hediyeleri (kalıcı perkler)

Sınırı **geçmekle** açılır (eşik/para değil), NG+'da kaybolmaz:

| Sınır | Hediye |
|---|---|
| Biyom 2 sonu | Bedenler %15 geç yorulur |
| Biyom 4 sonu | Dinç bedenler 1.5× daha sık |
| Biyom 6 sonu | Nefes +0.55 (0.4 yerine) ve çekim yarıçapı 9→12 |
| Biyom 8 sonu | Fenerler geçerken tam nefes verir |

### 4.5 Diğer

- **Dinç bedenler:** Spawn'ların bir kısmı ışıklı doğar (araba %25 — hızlı+cazip+riskli; kamyon %8 — engellere ölümsüz olduğu için ucuz ödül olmasın; diğerleri %10; biyom/hediye/sınır çarpanları uygulanır). Geçince tam stamina + 2.5 sn yorulmazlık.
- **Diriliş (rewarded ad simülasyonu):** Gece başına 1; buton: "Biri seni hatırladı — geri dön." Taşınan anlar korunur.
- **Kamyon garantisi:** (eski etap sisteminden kalan) kamyonla bariyer ezme = oyunun en tok aksiyonu; kamyon şansı KONVOY biyomunda 0.30.
- **Kayıt (v3):** `{bestRel, momsHome, spots, pile, seals, fin}` — rekor, eve giden anlar, tüketilen an noktaları, yolda bekleyen anlar, mühürlü fenerler, azat bayrağı.
- **Final sonrası:** "YENİDEN YÜRÜ" — yol/rekor/mühürler sıfırlanır, **ev ve anılar kalır**.

## 5. Ton ve Yazım İlkeleri

1. **Ekran, kaybettiğini değil seni bekleyeni gösterir.** Ölüm metni asla "başarısız" demez: "kedi seni bekliyor — yol hatırlıyor."
2. **Her kanca diegetik gerekçesini tek cümlede verebilmeli.** Streak sayacı yok → ruh parlar; slot sesi yok → yumuşak çan; revive → "biri seni hatırladı."
3. **Cüzdan dili yasak.** Anlar harcanmaz; yerleşir/bırakılır/eve gider.
4. **Final kutsal.** ŞAFAK biyomunda kompulsiyon makinesi (zincir, altın misafir) susar.
5. Canvas piksel fontu kısıtlı glif setine sahip (Türkçe diakritik yok; I, H, C, Z gibi harfler yok) — canvas'taki tüm kelimeler bu setten seçildi (AN, DURAK yerine EMANET, REKOR, EVE, MOLA→kaldırıldı vb.); serbest Türkçe yalnız DOM ekranlarında.

## 6. Tasarım Tarihçesi — Denenen ve Reddedilenler

Değerlendirici için önemli: aşağıdakiler denendi/tartışıldı ve **bilinçli reddedildi**. Aynı önerileri tekrarlamak yerine, reddin gerekçesinin yanlış olduğunu düşünüyorsanız onu söyleyin.

1. **Skor + combo + yıldız sistemi** → kesildi. Skor hiçbir yere bağlanmıyordu; combo "sürekli zıpla" diyerek oyunun gerçek gerilimiyle (doğru anı bekle) çelişiyordu.
2. **Para topla → evi süsle metası** → reddedildi (sahibi: "ruh oyununda para eğreti, amaçsız"). Teşhis: harcanamayan para ajans vermez; içinde kimse olmayan evi süslemek "mozole dekore etmek."
3. **Level/etap yapısı (her level sonunda ev + "EVDESİN")** → reddedildi. Varış her 60 saniyede bedavaya verilince amaç olmaktan çıkıyor; sayaçlar o boşluğu taşıyamıyor.
4. **10 etaplık sonlu, MOLA overlay'li rota** → güzeldi ama kompulsiyon üretmedi (sahibi: "10 etap bağımlı yapmaz"). Overlay, dürtü ile buton arasına giren her şeydir.
5. **Kalkan perki** → kaldırıldı: savunma perki gerilimi düşürüyor. Yerine cesaret ödülleri (clutch/zincir).
6. **"Kayıp ruhları kurtar" fantezisi** → reddedildi: bedeli olmayan kurtarma ahlaki ağırlık taşımaz; kurtarıcı fantezisi, yaşayan bedenleri eskitip atan çekirdek fiille çelişir.
7. **Checkpoint'ten bedava devam** → kaldırıldı: ölümün bedeli kompulsiyonun yakıtı; devam hakkı artık kazanılır (EMANET mühürü).

## 7. Bilinen Açık Konular (sahibin ve ekibin bildiği eksikler)

- **Denge elle test edilmedi:** gece biyomlarının (7–9) zorluk eğrisi, altın misafir sıklığı, EMANET fiyatı (3 an) — hepsi kâğıt üstünde makul, elde doğrulanmadı.
- **EMANET akışı** bot testinde tetiklenemedi; mantık gözden geçirildi ama gerçek elle oynanmadı.
- **Ölüm ekranı yol şeridi grafiği** (ölüm noktası ↔ kedi/bayrak konumunu çizgi üstünde gösteren mini harita) tasarlandı, henüz çizilmedi — şimdilik metin.
- **Onboarding:** yeni oyuncuya "her gece aynı yol + rekor" yapısını ilk 30 saniyede anlatan bir an yok (menü metni + rehber el var, yeterliliği belirsiz).
- **Ses/müzik** biyomlara göre değişmiyor; biyom geçişi görsel olarak ani (yumuşatma yok).
- **Günlük kanca yok** (daily seed / streak) — bilinçli ertelendi, prototip kapsamı dışı.

## 8. Değerlendiriciden İstenenler

Lütfen genel izlenim yerine şu sorulara odaklan:

1. **Kompulsiyon döngüsü:** Rekor izi + sınır bölgesi + Son Nefes bahsi + altın misafir dörtlüsü, 30–90 saniyelik run'larda gerçekten "bir gece daha" dedirtir mi? Döngüde eksik ya da fazla halka var mı?
2. **İlk 5 dakika:** Yeni oyuncu ilk 3 gecede bu yapının cazibesini anlar mı? Onboarding'de en kritik boşluk ne?
3. **EMANET ikilemi:** "3 anını fenere bırak, gece oradan başlasın ama ev o anları hiç görmesin" — bu takas anlamlı ve okunaklı mı? Fiyat (3) ve konumlar (biyom sınırları) doğru mu?
4. **Sonlu oyun + kompulsiyon gerilimi:** "Eve varış = döngüden azat" fikri kompulsiyon hedefiyle çelişiyor mu, yoksa onu güçlendiriyor mu? Final sonrası ("YENİDEN YÜRÜ") yeterli mi?
5. **Fiil derinliği:** Tek input (tap-to-possess) + Son Nefes bahsi, uzun vadede yeterli beceri tavanı sunar mı? Fiile eklenecek TEK şey ne olurdu?
6. **Ton:** "Kumarhane mekaniği + hüzünlü hayalet kurgusu" evliliği tutarlı mı, yoksa bir taraf diğerini boğuyor mu?
7. Reddedilenler listesinden (bölüm 6) geri alınmayı hak eden bir şey var mı — ve neden?
