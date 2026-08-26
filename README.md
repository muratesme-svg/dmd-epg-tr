# DMD IPTV PRO - Kendi Otomatik EPG Hattımız (Türkçe)

Bu klasör, DMD IPTV PRO'nun kendi, ücretsiz, açık kaynak kodlu ve **sürekli
güncel** bir Türkçe TV rehberi (EPG/XMLTV) kaynağına sahip olması için
hazırlandı. Üçüncü taraf sitelere (epgshare01.online gibi) bağımlı kalmak
yerine, açık kaynaklı [iptv-org/epg](https://github.com/iptv-org/epg)
projesini GitHub'ın kendi sunucularında (GitHub Actions), tamamen ücretsiz
olarak, günde bir kez otomatik çalıştırıp sonucu kendi deponuzda barındırır.

Sonuçta elde edeceğiniz adres (`https://raw.githubusercontent.com/...`)
büyük bir CDN (Fastly) üzerinden servis edilir - küçük/bağımsız EPG
sitelerinin sıkça karşılaştığı "bot sanılıp 403 ile reddedilme" sorununu
yaşama ihtimali çok daha düşüktür.

## Neden bunu siz kurmalısınız (ben otomatik yapamıyorum)

Bu iş bir GitHub hesabı ve o hesaba ait bir depo (repository) gerektiriyor.
Güvenlik nedeniyle sizin adınıza bir GitHub hesabı/deposu oluşturamam veya
kimliğinizle bir yere içerik yükleyemem - bu yüzden aşağıdaki adımları siz
(veya benimle ekran paylaşarak/adım adım yönlendirerek) tamamlamanız
gerekiyor. Tamamlandığında bana sonuç adresini iletmeniz yeterli, ben de
uygulamanın varsayılan EPG kaynağını o adrese güncellerim.

## Kurulum Adımları

1. **GitHub hesabı** - yoksa [github.com](https://github.com) üzerinden
   ücretsiz bir hesap açın.

2. **Yeni bir depo (repository) oluşturun** - sağ üstteki "+" > "New
   repository". İsim örneğin `dmd-epg-tr` olabilir. **Public** (herkese
   açık) seçin - raw.githubusercontent.com üzerinden ücretsiz erişim için
   deponun public olması gerekiyor (içinde gizli/kişisel hiçbir veri yok,
   sadece TV yayın rehberi verisi).

3. **Bu klasördeki dosyaları o depoya yükleyin** - en kolay yol: depo
   sayfasında "Add file" > "Upload files" ile bu `README.md` dosyasını ve
   `.github/workflows/epg-tr.yml` dosyasını (klasör yapısını koruyarak)
   sürükleyip bırakmak. (Git'e aşinaysanız elbette `git push` ile de
   yapabilirsiniz.)

4. **Actions'a yazma izni verin** - depo içinde Settings > Actions >
   General sekmesine gidin, en alttaki "Workflow permissions" bölümünden
   "Read and write permissions" seçeneğini işaretleyip kaydedin. (Bu,
   otomatik iş akışının ürettiği güncel rehber dosyasını depoya geri
   commit'leyebilmesi için gerekli.)

5. **İlk çalıştırmayı elle tetikleyin** - depo içinde "Actions" sekmesine
   gidin, soldan "EPG Turkiye Guncelle" iş akışını seçin, sağ üstten "Run
   workflow" düğmesine basın. İlk çalıştırma birkaç dakika sürebilir
   (yüzlerce kanal için veri çekiliyor).

6. **Sonucu kontrol edin** - iş akışı yeşil tik ile bittiğinde, depoda
   `dist/guide-tr.xml`, `dist/guide-tr.xml.gz`, `dist/manifest-tr.json` ve
   `dist/registry.json` dosyalarının oluştuğunu göreceksiniz. Kalıcı,
   herkese açık adresiniz şu şekilde olacak (KULLANICI_ADI ve REPO_ADI'nı
   kendi bilgilerinizle değiştirin):

   ```
   https://raw.githubusercontent.com/KULLANICI_ADI/dmd-epg-tr/main/dist/guide-tr.xml.gz
   ```

   Manifest adresi (küçük, hızlı kontrol dosyası - versiyon/hash/kanal
   sayısı içerir, uygulama önce bunu kontrol edip değişiklik yoksa büyük
   dosyayı hiç indirmeyebilir):

   ```
   https://raw.githubusercontent.com/KULLANICI_ADI/dmd-epg-tr/main/dist/manifest-tr.json
   ```

7. Bu adresi bana iletin - uygulamanın varsayılan (ve gerekirse tek)
   Türkçe EPG kaynağını bu adrese güncelleyeceğim. Kurulumdan sonra hattınız
   her gün 03:00 UTC'de (Türkiye saatiyle yaklaşık 06:00-07:00) kendini
   otomatik tazeler - hiçbir elle müdahale gerekmez.

## Bu neyi kapsıyor?

İş akışı, iptv-org/epg projesinin Türkçe (`lang=tr`) kanal verisi sunan
tüm site kaynaklarını (Digiturk, D-Smart, Türksat Kablo, TV+ dahil resmi
Türk yayıncı rehberleri ile birlikte) birleştirerek çekiyor - tek bir
siteye bağımlı değil. iptv-org/epg zamanla yeni kaynaklar eklerse,
`.github/workflows/epg-tr.yml` içindeki `--sites=...` listesini
güncelleyerek kapsamı genişletebilirsiniz (bu konuda da yardımcı olabilirim).

## Manifest ve registry (çoklu ülke/dil için hazır altyapı)

DMD IPTV PRO kaynak/kaynaklar (Xtream/M3U) değil, altyapı sunduğu için bu
hat şimdiden **tek ülkeye özel olmayacak şekilde** kuruldu:

- `manifest-tr.json` - bu rehberin versiyon bilgisi (üretilme zamanı, kanal
  ve program sayısı, sha256 özeti, boyutu). Uygulama önce bunu indirir;
  sha256 daha önce görülenle aynıysa büyük `.xml.gz` dosyasını hiç
  indirmez - gereksiz trafik ve ayrıştırma (parse) önlenir.
- `registry.json` - hangi ülke/dil rehberlerinin var olduğunu ve nerede
  olduğunu listeleyen tek giriş noktası. Şu an sadece `"tr"` girişi var;
  yarın `"de"`, `"fr"` gibi başka bir ülke eklendiğinde uygulama tarafı
  hiç değişmeden yeni girişi otomatik keşfedebilir.
- **Yeni bir ülke/dil eklemek**: `epg-tr.yml` dosyasını kopyalayıp
  `--sites=...`/`--lang=...` değerlerini ve `guide-tr` → `guide-de` gibi
  dosya adlarını değiştirmek yeterli; manifest/registry üretim mantığı
  aynı kalıyor. Bu adımı istediğinizde birlikte yapabiliriz.

## Dürüstlük notu

Bu, üçüncü taraf sitelere göre çok daha güvenilir ve şeffaf bir yöntem
(kaynak kodu açık, ne zaman çalıştığı ve ne çektiği tamamen görünür) - ama
yine de nihai veri kalitesi, iptv-org/epg'nin desteklediği kaynak
sitelerin kendi rehber verisinin doğruluğuna bağlı. Bazı kanallar için
veri eksik/gecikmeli olabilir; bu, elimizdeki en sağlam açık kaynaklı
seçenek olmakla birlikte %100 garanti değildir.
