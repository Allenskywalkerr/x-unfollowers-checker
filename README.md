# 🐦 Twitter (X) - Seni Takip Etmeyenler Bulucu ve Tek Tıkla Temizleme Aracı

Twitter (X) üzerinde seni geri takip etmeyen kullanıcıları tespit eden ve sayfadan ayrılmadan, tek tıkla takipten çıkmanı sağlayan tarayıcı konsolu (DevTools) tabanlı bir yardımcı araç.

X'in resmi API'si ücretli olduğu için bu araç tamamen **ücretsiz**, **kurulumsuz** ve tamamen kendi tarayıcında, kendi oturumunla çalışır. Hiçbir veri üçüncü bir sunucuya gönderilmez.

## ✨ Özellikler

- **Tam Otomatik Analiz:** `Following` ve `Followers` listelerini kendi kendine sırayla tarar ve karşılaştırır.
- **Doğru Filtreleme:** X'in araya sıkıştırdığı "Kimi takip etmeli" önerilerini otomatik eler; sadece gerçek takip edilen kişileri sayar.
- **Sayfa İçi Panel:** Analiz bitince ekranın sağında bir kontrol paneli açılır.
- **Tek Tıkla Takipten Çıkma:** Panelden, sayfa yenilemeden, doğrudan takipten çıkabilirsin.

## 🚀 Kullanım

1. [x.com](https://x.com) üzerinde hesabına giriş yap.
2. Kendi profilinden **"Following" (Takip Edilenler)** sayfasını aç:
   ```
   https://x.com/<kullanici_adin>/following
   ```
3. Tarayıcıda Geliştirici Araçları'nı aç:
   - Windows/Linux: `F12` veya `Ctrl+Shift+I`
   - Mac: `Cmd+Option+I`
4. **Console** sekmesine geç.
5. [`unfollowers.js`](./unfollowers.js) dosyasının tüm içeriğini kopyala, console'a yapıştır ve `Enter`'a bas.
6. Script otomatik olarak takip ettiklerini ve takipçilerini tarayıp karşılaştıracak; sonucu sağda açılan panelde göreceksin.
7. Panelden "Takipten Çık" butonuna basarak istediğin kişileri takipten çıkarabilirsin.

## ⚠️ Güvenlik ve Kullanım Uyarısı

**Önce şunu oku:** Bu script, X'in dokümante edilmiş resmi API'sini değil, web istemcisinin (x.com) arka planda kullandığı iç uç noktaları (internal/undocumented endpoints) kullanır. Bunun birkaç sonucu var:

- **ToS riski:** X'in Kullanım Şartları, otomatikleştirilmiş etkileşimleri (bulk takip/unfollow gibi) sınırlayabilir. Bu aracı kullanmak senin kendi sorumluluğundadır; kötüye kullanım hesabının geçici veya kalıcı kısıtlanmasına yol açabilir.
- **Kırılganlık:** X sık sık arayüzünü ve iç API yapısını değiştiriyor. Script bugün çalışsa da bir süre sonra bozulabilir; böyle bir durumda GitHub'da Issue açabilirsin.
- **Rate limit:** Takipten çıkma butonlarına art arda hızlı basma. Her tıklamadan sonra rengin değişmesini (✓ Çıkıldı) bekle ve 1-2 saniye ara ver. Günde 100-150'den fazla hesabı takipten çıkaracaksan işlemi gün içine yaymanı öneririz. Script "çok hızlı gidiyorsun" (429) durumunu ayrıca tespit edip sana bildirir.
- **Kod kaynağına dikkat:** Konsola yapıştırılan kodlar, o sekmedeki oturum bilgilerine (çerezler) erişebilir. Bu script'i **sadece bu resmi GitHub reposundan** al; başka yerlerde değiştirilmiş/kötü niyetli versiyonlarına karşı dikkatli ol.

## 🛠️ Nasıl Çalışır?

1. `getList()` fonksiyonu sayfayı belirli aralıklarla aşağı kaydırır, her adımda görünen kullanıcı kartlarından (`UserCell`) kullanıcı adlarını toplar ve "Kimi takip etmeli" bloklarını eler.
2. "Following" sayfasında, bir kartın gerçekten `unfollow` butonu içerip içermediği kontrol edilerek yanlış pozitifler tamamen elenir.
3. Sayfa artık kaydırılamıyorsa (aynı scroll pozisyonunda birkaç kez üst üste kalınırsa) taramanın bittiği anlaşılır.
4. Aynı işlem önce "Following", sonra "Followers" sekmesi için tekrarlanır.
5. İki liste karşılaştırılır: `following` içinde olup `followers` içinde olmayanlar = seni takip etmeyenler.
6. "Takipten Çık" butonuna basıldığında, tarayıcıdaki `ct0` (CSRF) çerezi okunur ve X'in web istemcisinin kullandığı `friendships/destroy.json` uç noktasına istek atılır — yani buton, senin manuel olarak yapacağın işlemi otomatikleştirir.

## 🩺 Sorun Giderme

| Belirti | Olası Neden / Çözüm |
|---|---|
| Panel hiç açılmıyor | Doğru sayfada (`/following`) olduğundan emin ol, sayfayı yenileyip tekrar dene. |
| Liste eksik/yanlış geliyor | X arayüz güncellemesi seçicileri bozmuş olabilir; Issue aç. |
| "Oturum Hatası!" | `ct0` çerezi bulunamadı; sayfayı yenileyip tekrar dene. |
| "⏸ Çok Hızlı! Bekle" | Rate limit'e takıldın; birkaç dakika bekleyip tekrar dene. |
| Tarama çok uzun sürüyor | Takipçi sayın fazlaysa normal; sekmeyi kapatmadan bekle. |

## 📄 Lisans

[MIT](./LICENSE) — istediğin gibi kullan, değiştir, paylaş, fork'la.

## 🤝 Katkıda Bulunma

X'in arayüzü değiştiğinde script bozulabilir. Bir sorun fark edersen Issue açmaktan veya PR göndermekten çekinme.
