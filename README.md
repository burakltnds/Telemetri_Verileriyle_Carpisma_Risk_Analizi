# Features
- LAT, LON         --> Enlem Boylam Koordinatları (Derece Cinsinden).
- SOG              --> Hız değeri (Knot Cinsinden).
- COG              --> Geminin izlediği yol (Derece Cinsinden).
- Heading          --> Geminin burnunun nere baktığı (Derece Cİnsinden).
- Status           --> Geminin mevcut durumu (Seferde, Demir atmış gibi).
- BaseDateTime     --> Telemetri verisinin gönderildiği tarih ve zaman.
- MMSI             --> Telemetri cihazının benzersiz kimlik numarası.
- IMO              --> Geminin benzersiz numarası.
- VesselName       --> Geminin adı.
- CallSign         --> Geminin benzersiz çağrı numarası.
- VesselType       --> Gemi Tipi.
- Length, Width    --> Geminin uzunluğu ve genişliği.
- Draft            --> Geminin su altında kalan kısmı.
- Cargo            --> Yük Tipi.
- TransceiverClass --> Telemetri Cihazı Tipi.
# EDA
- 14417 cihaz veri göndermiş(MMSI)
- LAT (+90 -90 arası olmalı) ve LON (+180 -180 arası olmalı)
- LAT ve LON değerleri her ne kadar üst maddedeki sınırlara uysada modelimizin kafasını karıştırabilecek outlier değerler bulunuyor
- SOG verisinde aşırı uç değerler vardır. Buda olası bir sensör arızası yada hata durumunu işaret ediyor bu veriler temizlenmeli. Ayrıca verinin yarısından fazlası 0'dan oluşmaktadır buda gemilerin hareket halinde olmadığını gösterir.Çarpışma risk analizi yaptığımız için bu gemiler verisetinden çıkarılmalıdır.
- COG verisi gayet temiz gözüküyor. yalnız bu veri açı değerini ifade ettiğinden modelin doğru bir çıkarım yapabilmesi için sinus ve kosinus dönüşümü yapmak mantıklı olacaktır
- Heading veriside COG gibi açısal bir değerdir fakat bu veride 4 milyona yakın 511 değeri vardır. 511 aslında bir açı değeri değil bir hata kodudur.
- Status, VesselType, Cargo asıl olarak encode edilmiş kategorik değerlerdir.
- Length ve Width verilerinde aykırı üst değerler var gibi durmasına karşın gerçek hayattaki çok büyük gemileri ifade ederler
- Draft verisinde negatif değerler sensör hatasıdır geminin su altında kalan kısmı negatif olamaz. boxplottaki üst değerlerse gerçek hayattaki büyük gemilerdir aykırı değer olarak değerlendirilemez
# Outlier
- Koordinat verilerinin LAT ve LON olması gereken aralıkta olup olmadığı kontrol edilmiştir.
- SOG verisi 0 ve 80 knot arasına sabitlenmiştir. 0 olanlar hareketsizliği belirttiği için kaldırılmıştır. Bu filtreleme sonrası elimizdeki 7 milyondan fazla verinin büyük kısmı kaybedilsede yine elimizde 2.6 milyondan fazla veri kalmıştır. 
- COG verisi 0-360 derece arasına sabitlendi.
- Heading Verisindeki 511 değerleri hatayı ifade ettiği için silinip yerine COG değerleriyle doldurulmuştur.bu iki değer akıntı rüzgar gibi etkenlerin olmadığı yerde aynı olmalıdır.rüzgar ve akıntının olmadığı varsayılarak doldurma işlemi gerçekleştirilmiştir.Aynı zamanda 360'dan büyük değerler kaldırılmıştır
- Draft değerinin 0'dan küçük olması engellenmiştir.
- Length ve Width değerlerindeki 0'lara boş değer atanmıştır. 5 metre'den kısa ve 2 metre'den dar olan küçük deniz araçları verisetinden kaldırılmıştır
- Draft ve Cargo verilerinin yüzde 60 - 70'e yakını eksik olduğundan bu metrikler kaldırılmıştır.
# Missing Value
- MMSI numarası ile gemi ismi arasında tutarsızlık olup olmadığı kontrol edilmiştir.
- MMSI, CallSign, IMO, VesselName değerleri aynı şeyi ifade ettiğinden eksik verisi olmayan MMSI değeri tutulmuştur, diğer değerler kaldırılmıştır.
- Length ve Width değerleri MMSI değerine göre gruplayarak doldurulmaya çalışılmıştır fakat bu yöntem işe yaramadığından ve elimizde yeteri kadar veri olduğundan bu değerleri boş olan satırlar silinmiştir.
- VesselType değeri yine MMSI değerine göre gruplayarak doldurulmaya çalışılmıştır fakat bu yöntem bu değer içinde geçersizdir.Modele çok fazla katkısının olmayacağını düşündüğüm için Status ve VesselType değerlerini kaldırdım.
- Bu işlemler sonucunda veri 1.6 milyona kadar düşmüştür
# Feature Engineering
- LAT ve LON verilerine uyguladığımız yuvarlama işlemiyle bir bakıma denizi parsellere bölmüş olduk bu parseler gemileri eşleştirirken işlemci gücünü korumamızı sağlayacak eğer bunu uygulamasaydık 1.6 milyon veriyi birbiri ile ikili eşleştirirken işlemci gücü ve ram bakımından sorun yaşayacaktık ızgara sayesinde ondalıklı basamağın ilk ikisini alarak denizi yaklaşık 1 km karelik alanlara bölmüş olduk.LatGrid ve LonGrid değerlerini oluşturduk.
- Gemilerin AIS cihazları sinyalleri sabit bir zamanda değil rastgele bir zamanda gönderir bu yüzden saat verisini 5 dakika kadar yuvarlayarak TimeGrid değerini oluşturduk ve ayrıca bu veri içinde yukardaki maddede olduğu gibi zamanı 5 dakikalık periyotlara böldük.Gemileri birbirleri ile eşlerken LatGrid + LonGrid + TimeGrid verilerini kullanarak işimizi kolaylaştıracağız.
- Heading ve COG verilerine sinus ve cosinus dönüşümü yapılmıştır bu sayede makine öğrenmesi modelimiz bu verilerin açısal bir değer olduğunu anlayabilecek.
- Tek 1 veri satırı gönderen gemileri matematisel hesaplamalarının yapılamayacağından dolayı (yer değiştirme, manevra kabiliyeti, ivme gibi) verisetinden kaldırılmıştır.
- Verileri MMSI numarası ve daha sonra BaseDateTime değerine göre sıraladım.
- Daha sonra MMSI numaralarına göre grupladığımız gemilerin zaman değişimi ve hız değişiminden yararlanarak ivme'yi buldum
- Aynı şekilde rota'daki değişime ve zaman farkına göre göre dönüş hızı hesaplanmıştır
- Ürettiğimiz yeni TimeGrid, LatGrid ve LonGrid metriklerine göre aynı zaman ve mekan ızgarası içinde olan gemilerin eşlendiği yeni bir dataframe oluşturulmuştur.Yaklaşık 750.000 çift oluşmuştur.
## **TCPA VE DCPA OLUŞTURULMASI**
- Daha sonra eşleşmiş iki gemi arası enlem ve boylam farkları derece cinsinden metre cinsine çevrilmiştir
- Knot cinsinden olan hız m/s cinsine çevrilmiştir
- trigonometrik fonksiyonlarla gemi hızı hız vektörüne çevirilmiştir.
- X ve Y eksenlerinde bağıl hızın karesi toplanmıştır ve eğer 0 ise bölme hatası almamak için çok küçük bi değere atanmıştır
- **TCPA** Gemilerin mevcut konumları ile hız vektörlerinin iç çarpımı kullanılarak, iki geminin birbirine en yakın konuma gelmesi için gereken süreyi verir. Eğer sonuç negatif çıkarsa, gemiler birbirine yaklaşmıyor, aksine birbirinden uzaklaşıyor demektir.
- **DCPA** TCPA süresi hıza ve mesafeye tekrar çarpılarak, gemiler o en yakın noktaya vardıklarında aralarında kalacak minimum mesafeyi hesaplar. Denizcilikte DCPA 0'a ne kadar yakınsa, çarpışma riski o kadar yüksektir demektir.
- DCPA değeri kullanılarak risk seviyesi Güvenli, Riskli, Çok Riskli şeklinde gruplandırılmıştır.
# Encoding
- Zaten kategorik veriler sayı şeklinde verildiği için tekrardan encode yapmaya gerek yoktur.
# Scaling 
- Ağaç tabanlı modeller kullanılacağı için ve sonradan oluşturulmuş açısal değerleri kaybetmemek için scaling uygulanmamıştır.
# Train Test Split
- En son model için gereksiz olan ve hedef verimize kopya olabilecek sutunları verisetinden çıkardım.
- Preprocess adımları sonucunda elimizde kalan 750.000 satır veriyi yüzde 20 Test, yüzde 80 Train olarak ayırdım.
