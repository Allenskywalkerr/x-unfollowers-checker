🐦 Twitter (X) - Seni Takip Etmeyenler Bulucu ve Tek Tıkla Temizleme Aracı

Twitter (X) üzerinde seni geri takip etmeyen kullanıcıları tespit eden ve sayfadan ayrılmadan, tek tıkla takipten çıkmanı sağlayan tarayıcı konsolu (DevTools) tabanlı bir yardımcı araç.

X'in resmi API'si ücretli olduğu için bu araç tamamen ücretsiz, kurulumsuz ve tamamen kendi tarayıcında, kendi oturumunla çalışır. Hiçbir veri üçüncü bir sunucuya gönderilmez.

✨ Özellikler


Tam Otomatik Analiz: Following ve Followers listelerini kendi kendine sırayla tarar ve karşılaştırır.
Doğru Filtreleme: X'in araya sıkıştırdığı "Kimi takip etmeli" önerilerini otomatik eler; sadece gerçek takip edilen kişileri sayar.
Sayfa İçi Panel: Analiz bitince ekranın sağında bir kontrol paneli açılır.
Tek Tıkla Takipten Çıkma: Panelden, sayfa yenilemeden, doğrudan takipten çıkabilirsin.


🚀 Kullanım


x.com üzerinde hesabına giriş yap.
Kendi profilinden "Following" (Takip Edilenler) sayfasını aç:


   https://x.com/<kullanici_adin>/following


Tarayıcıda Geliştirici Araçları'nı aç:

Windows/Linux: F12 veya Ctrl+Shift+I
Mac: Cmd+Option+I



Console sekmesine geç.
Aşağıdaki kod bloğunun sağ üst köşesindeki kopyala ikonuna tıkla (veya unfollowers.js dosyasını aç), console'a yapıştır ve Enter'a bas.
Script otomatik olarak takip ettiklerini ve takipçilerini tarayıp karşılaştıracak; sonucu sağda açılan panelde göreceksin.
Panelden "Takipten Çık" butonuna basarak istediğin kişileri takipten çıkarabilirsin.


📋 Hızlı Kopyala

<details>
<summary><b>Kodu görmek ve kopyalamak için tıkla</b></summary>
javascript/**
 * Twitter (X) - Seni Takip Etmeyenler Bulucu + Tek Tıkla Takipten Çık
 * ---------------------------------------------------------------------
 * Takip ettiğin ama seni geri takip etmeyen kullanıcıları bulur ve
 * paneldeki butonlarla tek tıkla takipten çıkmanı sağlar.
 *
 * KULLANIM:
 * 1. X (Twitter) hesabınla giriş yap.
 * 2. Kendi profiline git ve "Following" (Takip Edilenler) sayfasını aç:
 *    https://x.com/<kullanici_adin>/following
 * 3. Tarayıcıda F12 / Cmd+Option+I ile Developer Tools'u aç, "Console" sekmesine geç.
 * 4. Bu dosyanın tüm içeriğini kopyala, console'a yapıştır ve Enter'a bas.
 * 5. Script otomatik olarak Following ve Followers listelerini tarayıp
 *    ekranda seni takip etmeyenlerin listesini gösterecek.
 * 6. "Takipten Çık" butonlarına art arda hızlı basma; her tıklamadan sonra
 *    1-2 saniye bekle (rate-limit / geçici kısıtlama riskini azaltır).
 *
 * ÖNEMLİ: Bu script X'in resmi/dokümante edilmiş API'sini değil, web
 * istemcisinin kullandığı iç uç noktaları (internal endpoints) kullanır.
 * Bu nedenle X'in arayüz güncellemelerinde bozulabilir ve kullanımı X'in
 * Hizmet Şartları'na aykırı sayılabilir. Kullanım sorumluluğu kullanıcıya aittir.
 */

(async function () {
    // 1. Doğru sayfada mıyız kontrolü
    if (!window.location.href.includes('/following')) {
        alert("HATA: Lütfen bu kodu SADECE 'Takip Edilenler' (Following) sayfasındayken başlatın!");
        return;
    }

    console.log("%c🔥 Kusursuz Tarama Başlatıldı! Lütfen bekleyin...", "color: #1da1f2; font-size: 16px; font-weight: bold;");

    // 2. Sayfayı kaydırma ve kullanıcı toplama fonksiyonu
    async function getList(isFollowingPage) {
        let tracked = new Set();
        let lastScrollY = window.scrollY;
        let stuckCount = 0;

        return new Promise((resolve) => {
            let timer = setInterval(() => {
                let primaryColumn = document.querySelector('[data-testid="primaryColumn"]');
                if (primaryColumn) {
                    primaryColumn.querySelectorAll('[data-testid="UserCell"]').forEach(el => {
                        // "Kimi takip etmeli" / "Who to follow" önerilerini yoksay
                        let isSuggested = el.closest('[aria-label="Kimi takip etmeli"]') || el.closest('[aria-label="Who to follow"]') || el.closest('aside');
                        if (isSuggested) return;

                        let links = el.querySelectorAll('a[role="link"]');
                        let username = "";
                        for (let link of links) {
                            let text = link.innerText || "";
                            if (text.startsWith('@')) {
                                username = text;
                                break;
                            }
                        }

                        if (username) {
                            if (isFollowingPage) {
                                // "Following" sayfasındaysak, kartın gerçekten bir "unfollow"
                                // butonu içerdiğinden emin ol (önerileri %100 eler)
                                let unfollowBtn = el.querySelector('[data-testid$="-unfollow"]');
                                if (unfollowBtn) tracked.add(username);
                            } else {
                                tracked.add(username);
                            }
                        }
                    });
                }

                console.log(`[Tarama] Şu an toplanan: ${tracked.size} kişi`);
                window.scrollBy(0, 900);

                let currentScrollY = window.scrollY;
                if (currentScrollY === lastScrollY) {
                    stuckCount++;
                } else {
                    stuckCount = 0;
                    lastScrollY = currentScrollY;
                }

                // Sayfa 12 denemede (yaklaşık 9-10 saniye) aşağı inemiyorsa sonuna gelmişizdir
                if (stuckCount >= 12) {
                    clearInterval(timer);
                    resolve(Array.from(tracked));
                }
            }, 800);
        });
    }

    // 3. Takip Edilenleri (Following) Tara
    console.log("%cAdım 1/2: Takip ettikleriniz taranıyor...", "color: #ffcc00; font-weight: bold;");
    let following = await getList(true);
    console.log(`✅ Takip edilen ${following.length} kişi bulundu.`);

    if (following.length === 0) {
        alert("Kimse bulunamadı! Sayfanın yüklendiğinden emin olup tekrar deneyin.");
        return;
    }

    // 4. Takipçiler (Followers) sekmesine geçiş yap
    console.log("%cAdım 2/2: Takipçiler sekmesine geçiliyor...", "color: #ffcc00; font-weight: bold;");
    window.scrollTo(0, 0); // Üste çık ki butonlar yüklensin
    await new Promise(r => setTimeout(r, 1000)); // 1 saniye bekle

    let followersTab = document.querySelector('a[href$="/followers"]');
    if (!followersTab) {
        alert("HATA: 'Takipçiler' sekmesi bulunamadı. Lütfen sayfayı yenileyip tekrar deneyin.");
        return;
    }

    followersTab.click(); // Takipçiler butonuna tıkla
    await new Promise(r => setTimeout(r, 3500)); // Sayfanın değişmesi ve yeni listenin yüklenmesi için bekle

    // 5. Takipçileri (Followers) Tara
    let followers = await getList(false);
    console.log(`✅ Takipçi ${followers.length} kişi bulundu.`);

    // 6. Karşılaştırma yap
    let nonBackFollowers = following.filter(user => !followers.includes(user));

    // 7. Ekranda Butonlu Arayüz (UI) Oluştur
    let ui = document.createElement('div');
    ui.style.cssText = "position:fixed; top:70px; right:30px; width:400px; max-height:80vh; overflow-y:auto; background-color:#15202b; border:2px solid #1da1f2; border-radius:15px; z-index:999999; padding:20px; color:#ffffff; box-shadow: 0 4px 15px rgba(0,0,0,0.8); font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;";

    let closeBtn = document.createElement('button');
    closeBtn.innerText = "Kapat X";
    closeBtn.style.cssText = "background:#ff4a4a; color:white; border:none; padding:5px 10px; cursor:pointer; float:right; border-radius:5px; font-weight:bold;";
    closeBtn.onclick = () => ui.remove();

    let title = document.createElement('h3');
    title.innerText = `Seni Takip Etmeyenler (${nonBackFollowers.length})`;
    title.style.cssText = "margin-top:0; font-size:18px; border-bottom: 1px solid #38444d; padding-bottom: 10px;";

    let info = document.createElement('p');
    info.innerText = "Takipten çıkarken butonlara art arda hızlı basma, aralarda 1-2 sn bekle.";
    info.style.cssText = "font-size:12px; color:#8899a6;";

    let list = document.createElement('ul');
    list.style.cssText = "padding: 0; list-style: none; margin: 0;";

    // Takipten Çıkma API Fonksiyonu
    window.directUnfollow = async function (btn, username) {
        try {
            btn.innerText = "⏳...";
            btn.disabled = true;
            btn.style.opacity = "0.7";

            // X'in web istemcisinin kullandığı yetkilendirme bilgilerini tarayıcıdan oku
            const getCookie = (name) => document.cookie.match('(^|;)\\s*' + name + '\\s*=\\s*([^;]+)')?.pop() || '';
            const csrfToken = getCookie('ct0');
            // Not: Bu, X'in web istemcisinin kullandığı genel (herkese açık) bearer token'ıdır.
            const bearer = "Bearer AAAAAAAAAAAAAAAAAAAAANRILgAAAAAAnNwIzUejRCOuH5E6I8xnZz4puTs%3D1Zv7ttfk8LF81IUq16cHjhLTvJu4FA33AGWWjCpTnA";

            if (!csrfToken) {
                btn.innerText = "Oturum Hatası!";
                btn.style.backgroundColor = "#e0245e";
                btn.disabled = false;
                console.warn("ct0 çerezi bulunamadı. Sayfayı yenileyip tekrar dene.");
                return;
            }

            let cleanName = username.replace('@', '');

            let res = await fetch('https://x.com/i/api/1.1/friendships/destroy.json', {
                method: 'POST',
                headers: {
                    'authorization': bearer,
                    'x-csrf-token': csrfToken,
                    'content-type': 'application/x-www-form-urlencoded'
                },
                body: `screen_name=${cleanName}`
            });

            if (res.ok) {
                btn.innerText = "✓ Çıkıldı";
                btn.style.backgroundColor = "#17bf63";
                btn.style.color = "white";
                btn.style.opacity = "1";
            } else if (res.status === 429) {
                // Rate limit: X "çok hızlı gidiyorsun" diyor, ayrı ve net bir mesaj göster
                btn.innerText = "⏸ Çok Hızlı! Bekle";
                btn.style.backgroundColor = "#ffcc00";
                btn.style.color = "#15202b";
                btn.disabled = false;
                console.warn(`Rate limit! "${username}" için biraz bekleyip tekrar dene.`);
            } else {
                btn.innerText = "Hata!";
                btn.style.backgroundColor = "#e0245e";
                btn.disabled = false;
            }
        } catch (err) {
            btn.innerText = "Hata!";
            btn.style.backgroundColor = "#e0245e";
            btn.disabled = false;
            console.error("Takipten çıkma hatası:", err);
        }
    };

    nonBackFollowers.forEach(user => {
        let cleanName = user.replace('@', '');
        let li = document.createElement('li');
        li.style.cssText = "margin: 12px 0; display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #2f3336; padding-bottom: 8px;";

        let linkSpan = document.createElement('span');
        linkSpan.innerHTML = `<a href="https://x.com/${cleanName}" target="_blank" style="color:#1da1f2; text-decoration:none; font-weight:bold; font-size: 15px;">🔗 ${user}</a>`;

        let actionBtn = document.createElement('button');
        actionBtn.innerText = "Takipten Çık";
        actionBtn.style.cssText = "background-color: transparent; border: 1px solid #ff4a4a; color: #ff4a4a; padding: 5px 12px; border-radius: 999px; font-weight: bold; cursor: pointer; transition: 0.2s;";
        actionBtn.onmouseover = () => { if (!actionBtn.disabled) { actionBtn.style.backgroundColor = "rgba(255, 74, 74, 0.1)"; } };
        actionBtn.onmouseout = () => { if (!actionBtn.disabled) { actionBtn.style.backgroundColor = "transparent"; } };

        actionBtn.onclick = () => window.directUnfollow(actionBtn, user);

        li.appendChild(linkSpan);
        li.appendChild(actionBtn);
        list.appendChild(li);
    });

    ui.appendChild(closeBtn);
    ui.appendChild(title);
    ui.appendChild(info);
    ui.appendChild(list);
    document.body.appendChild(ui);

    console.clear();
    console.log("%c🎉 İŞLEM TAMAMLANDI!", "color: #00ff00; font-weight: bold; font-size: 18px;");
    console.log(`Takip Ettiğin: ${following.length}`);
    console.log(`Seni Takip Eden: ${followers.length}`);
    console.log(`Seni Takip Etmeyen: ${nonBackFollowers.length}`);
})();

</details>
⚠️ Güvenlik ve Kullanım Uyarısı

Önce şunu oku: Bu script, X'in dokümante edilmiş resmi API'sini değil, web istemcisinin (x.com) arka planda kullandığı iç uç noktaları (internal/undocumented endpoints) kullanır. Bunun birkaç sonucu var:


ToS riski: X'in Kullanım Şartları, otomatikleştirilmiş etkileşimleri (bulk takip/unfollow gibi) sınırlayabilir. Bu aracı kullanmak senin kendi sorumluluğundadır; kötüye kullanım hesabının geçici veya kalıcı kısıtlanmasına yol açabilir.
Kırılganlık: X sık sık arayüzünü ve iç API yapısını değiştiriyor. Script bugün çalışsa da bir süre sonra bozulabilir; böyle bir durumda GitHub'da Issue açabilirsin.
Rate limit: Takipten çıkma butonlarına art arda hızlı basma. Her tıklamadan sonra rengin değişmesini (✓ Çıkıldı) bekle ve 1-2 saniye ara ver. Günde 100-150'den fazla hesabı takipten çıkaracaksan işlemi gün içine yaymanı öneririz. Script "çok hızlı gidiyorsun" (429) durumunu ayrıca tespit edip sana bildirir.
Kod kaynağına dikkat: Konsola yapıştırılan kodlar, o sekmedeki oturum bilgilerine (çerezler) erişebilir. Bu script'i sadece bu resmi GitHub reposundan al; başka yerlerde değiştirilmiş/kötü niyetli versiyonlarına karşı dikkatli ol.


🛠️ Nasıl Çalışır?


getList() fonksiyonu sayfayı belirli aralıklarla aşağı kaydırır, her adımda görünen kullanıcı kartlarından (UserCell) kullanıcı adlarını toplar ve "Kimi takip etmeli" bloklarını eler.
"Following" sayfasında, bir kartın gerçekten unfollow butonu içerip içermediği kontrol edilerek yanlış pozitifler tamamen elenir.
Sayfa artık kaydırılamıyorsa (aynı scroll pozisyonunda birkaç kez üst üste kalınırsa) taramanın bittiği anlaşılır.
Aynı işlem önce "Following", sonra "Followers" sekmesi için tekrarlanır.
İki liste karşılaştırılır: following içinde olup followers içinde olmayanlar = seni takip etmeyenler.
"Takipten Çık" butonuna basıldığında, tarayıcıdaki ct0 (CSRF) çerezi okunur ve X'in web istemcisinin kullandığı friendships/destroy.json uç noktasına istek atılır — yani buton, senin manuel olarak yapacağın işlemi otomatikleştirir.


🩺 Sorun Giderme

BelirtiOlası Neden / ÇözümPanel hiç açılmıyorDoğru sayfada (/following) olduğundan emin ol, sayfayı yenileyip tekrar dene.Liste eksik/yanlış geliyorX arayüz güncellemesi seçicileri bozmuş olabilir; Issue aç."Oturum Hatası!"ct0 çerezi bulunamadı; sayfayı yenileyip tekrar dene."⏸ Çok Hızlı! Bekle"Rate limit'e takıldın; birkaç dakika bekleyip tekrar dene.Tarama çok uzun sürüyorTakipçi sayın fazlaysa normal; sekmeyi kapatmadan bekle.

📄 Lisans

MIT — istediğin gibi kullan, değiştir, paylaş, fork'la.

🤝 Katkıda Bulunma

X'in arayüzü değiştiğinde script bozulabilir. Bir sorun fark edersen Issue açmaktan veya PR göndermekten çekinme.
