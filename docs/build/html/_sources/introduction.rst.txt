============================================
Temel Kavramlar ve İşletim Sistemi Yapıları
============================================

Kursumuzun başında iki hafta bazı temel kavramlar ve konular üzerinde duracağız.

Sistem Programlama Kavramı
==========================

Bilgisayar donanımıyla arayüz oluşturan, uygulama programlarına çeşitli bakımlardan hizmet veren programlara
*sistem programları*, programlamanın bunlarla ilgili alanına da *sistem programlama (system programming)*
denilmektedir. Sistem programlama etkinlikleri aşağı seviyeli olma eğilimindedir. Bunları yazmak için önemli ölçüde
teorik bilgiye ve uygulama becerisine gereksinim duyulmektedir. Sistem programlama *programlamanın yükte hafif pahada
ağır* bir alanını oluşturmaktadır. Bu yönüyle adeta yazılımın ağır sanayisi niteliğindedir. Bilişim sektöründeki
Microsoft, Apple, Google gibi pek manyak büyük kurum geliştirdikleri sistem yazılımları sayesinde bu hale
gelmişlerdir. Tipik sistem programlama uygulamalarından bazıları şunlardır:

* İşletim Sistemleri
* Derleyiciler ve Yorumlayıcılar
* Editörler
* Gömülü Sistem Uygulamaları
* Debug Programları
* Aşağı Seviyeli Haberleşme Programları
* Virüs ve Antivirüs Yazılımları
* Çevre Birimlerinin ve Diğer Donanımsal Aygıtların Programlanması ve Aygıt Sürücüleri
* Veritabanı Motorları
* Sanallaştırma Yazılımları ve Emülatör Yazılımları
* Oyun Motorları
* ...

Sistem programlama etkinlikleri için en çok kullanılan programlama dilleri C, C++ ve Sembolik Makine Dilleridir.
Rust Programlama Dili de son yıllarda bu alanda bir yer edinmeye çalışmaktadır. Her ne kadar sistem programlama
denildiğinde akla C, C++, Rust ve sembolik makine dilleri geliyorsa da bazı sistem programları Java ve C# gibi yüksek
seviyeli dillerle de yazılabilmektedir.

İşletim Sistemleri ve Kaynak Yönetimi
====================================

İşletim sistemleri bilgisayar donanımının kaynaklarını yöneten, bilgisayar donanımı ile kullanıcı arasında arayüz
oluşturan sistem programlarıdır. Bilgisayar bilimlerinin akademik öncülerinin çoğu işletim sistemlerini bir kaynak
yöneticisi *resource manager* olarak tanımlamıştır. Uygulama programları işletim sisteminin sağladığı olanaklardan
faydalanmaktadır.

.. image:: _static/isletim_sistemi_katmanlari.png
   :alt: İşletim Sistemi Katmanları
   :align: center

*programın çıktısı gibidir*

İşletim sistemlerinin yönettigi kaynakların en önemlileri şunlardır:

CPU
  İşletim sistemi hangi programın ne zaman, ne kadar süre için CPU'ya atanacağına karar verip bu işlemleri
  gerçekleştirmektedir.

Ana Bellek (Main Memory (RAM))
  İşletim sistemi programların ana belleğin neresine yükleneceğine karar verir ve ana bellek kullanımını düzenler.

İkincil Bellekler
  İşletim sistemi bir dosya sistemi *file system* oluşturarak dosyaların parçalarını ikincil belleklerde etkin bir
  biçimde tutar ve kullanıcılara bir dosya kavramıyla sunar.

Çevre Birimleri (klavye, fare, yazıcı vs.)
  İşletim sistemi fare, klavye, yazıcı gibi çevre birimlerini yöneterek onları kullanıma hazır hale getirir.
  Yardımcı işlemcileri (denetleyicileri) programlayarak onların işlev görmesini sağlamaktadır.

Ağ İşlemleri
  İşletim sistemi ağa ilişkin donanım birimlerini yöneterek ve çeşitli ağ protokollerini oluşturarak dışarıdan gelen
  bilgileri onları talep eden programlara iletir.

İşletim sistemleri kaynak yönetimine göre alt sistemlere ayrılarak da incelenebilmektedir. Örneğin işletim sisteminin
*çizelgeleyici (scheduler)* alt sistemi demekle CPU yönetimini sağlayan alt sistemi kastedilmektedir. Ana bellek
yönetimi *memory management* yine soyutlanarak incelenen önemli alt sistemlerden biridir. İşletim sistemlerinin
ikincil bellek yönetimine *dosya sistemi (file system)* da denilmektedir. Tabii bütün bu sistemler birbirinden kopuk
olarak değil birbirleriyle ilişkili bir biçimde işlev görmektedir. Bu durumu insanın *solunum sistemi*, *dolaşım
sistemi*, *sinir sistemi*, *boşaltım sistemi* gibi alt sistemlerine benzetebiliriz. Bu alt sistemlerin birinde bile
çalışma bozukluğu oluşsa insan yaşamını yitirebilmektedir.

İşletim sistemleri yapı olarak iki kısımdan oluşmaktadır: Çekirdek *kernel* ve kabuk *shell*. Çekirdek işletim
sisteminin donanımı kontrol eden ve kaynakları yöneten motor kısmıdır. Aslında işletim sistemi denildiğinde akla
çekirdek gelmektedir. Kabuk ise işletim sisteminin kullanıcı ile arayüz oluşturan önyüzüdür. Örneğin UNIX/Linux
sistemlerinde *bash* gibi komut satırı, GNOME, KDE gibi pencere yöneticileri, Windows'taki masaüstü (Explorer),
macOS'teki masaüstü (Aqua) bu işletim sistemlerinin kabuk kısımlarını oluşturmaktadır.

.. image:: _static/cekirdek_kabuk_yapisi.png
   :alt: Çekirdek Kabuk Yapısı
   :align: center

*programın çıktısı gibidir*

Peki işletim sistemi bu kadar temel donanım yönetimini sağlıyorsa işletimi olmadan programlama yapılabilir mi?
İşletim sistemi olmadan programlama faaliyetine halk arasında *bare metal programlama* denilmektedir. Bare metal
programlama genellikle mikrodenetleyicilerin kullanıldığı gömülü sistemlerde uygulanmaktadır. Bare metal
programlama özel bir amaca hizmet edecek biçimde yapılmaktadır. Amaçlar fazlalaştığı zaman ve sistem karmaşıklaştığı
zaman artık işletim sistemlerine gereksinim duyulmektedir.

Bazı kontrol yazılımları işletim sistemlerinin bazı etkinliklerini de sağlamaktadır. Bir kontrol yazılımının
işletim sistemi olarak isimlendirilmesi için yukarıda açıkladığımız kaynak yönetimlerinin önemli bir bölümününü
sağlıyor olması gerekir. Bu kaynak yönetimlerinin çoğunu sağlamayan kontrol yazılımlarına genel olarak *firmware*
de denilmektedir.

İşletim Sistemlerinin Sınıflandırılması
=======================================

İşletim sistemleri çeşitli biçimlerde sınıflandırılabilmektedir:

Proses Yönetimine Göre
  Aynı anda tek bir programı çalıştıran işletim sistemlerine *tek prosesli (single processing)*, aynı anda birden
  fazla programı çalıştırabilen işletim sistemlerine ise *çok prosesli (multiprocessing)* işletim sistemleri
  denilmektedir. Örneğin DOS işletim sistemi tek prosesli bir sistemdi. Biz bu işletim sisteminde bir programı
  çalıştırırdık, ancak çalıştırdığımız program sonlanınca başka bir programı çalıştırabilirdik. Halbuki Windows,
  UNIX/Linux, MacOS gibi işletim sistemleri çok prosesli işletim sistemleridir.

Kullanıcı Sayısına Göre
  Birden fazla farklı kullanıcının çalışabildiği sistemlere *çok kullanıcılı (multiuser)*, tek bir kullanıcının
  çalışabildiği sistemlere *tek kullanıcılı (single user)* sistemler denilmektedir. Genellikle çok prosesli işletim
  sistemleri aynı zamanda çok kullanıcılı sistemlerdir. Birden fazla kullanıcının söz konusu olduğu sistemlerde
  kullanıcıların yetkilerinin ayarlanması, kullanıcıların birbirlerinin alanlarına erişmesinin engellenmesi, sistem
  kaynaklarını belli bölüşmesi gerekebilmektedir. Örneğin DOS tek kullanıcılı bir sistemdi. Halbuki Windows,
  UNIX/Linux ve macOS sistemleri çok kullanıcılı sistemlerdir.

Çekirdek Yapısına Göre
  İşletim sistemleri çekirdek yapısına göre *tek parçalı çekirdekli (monolithic kernel)* ve *mikro çekirdekli
  (microkernel)* olmak üzere ikiye ayrılmaktadır. Tek parçalı çekirdekli işletim sisteminin büyük kısmı çekirdek
  modunda çalışır. Mikro çekirdekli sistemlerde ise çekirdek modunda çalışan kısım minimize edilmeye çalışılmıştır.
  Aslında tek parçalı ve mikro çekirdekli tasarımları bir spektrum olarak düşünebiliriz. (Örneğin bu spektrumda bazı
  çekirdekler tek parçalı tarafa yakın bazıları ise mikro tarafa yakın olabilmektedir.) Linux çekirdeği tek parçalı
  (monolithic) tarafa daha yakındır.

Mikro Çekirdekli Sistem Örnekleri
---------------------------------

+-----------------+--------------------------+---------------------------+-----------------------------+
| Sistem          | Kategori                 | Kullanım Alanı            | Olgunluk                    |
+=================+==========================+===========================+=============================+
| MINIX 3         | Saf Mikro Kernel         | Eğitim / Araştırma        | Akademik                    |
+-----------------+--------------------------+---------------------------+-----------------------------+
| QNX             | Saf Mikro Kernel         | Gömülü / Gerçek Zamanlı   | Üretim (Production)         |
+-----------------+--------------------------+---------------------------+-----------------------------+
| seL4            | Saf Mikro Kernel         | Güvenlik Kritik           | Üretim (Formal Verified)    |
+-----------------+--------------------------+---------------------------+-----------------------------+
| Mach            | Saf Mikro Kernel         | Araştırma / Tarihsel      | Tarihi Referans             |
+-----------------+--------------------------+---------------------------+-----------------------------+
| GNU Hurd        | Mach Tabanlı             | Genel Amaçlı              | Geliştirme Aşamasında       |
+-----------------+--------------------------+---------------------------+-----------------------------+
| macOS/iOS (XNU) | Mach Tabanlı (Hibrit)    | Masaüstü / Mobil          | Üretim (Hibrit Tasarım)     |
+-----------------+--------------------------+---------------------------+-----------------------------+
| L4 / Fiasco     | Akademik / Araştırma     | Araştırma / Gömülü        | Araştırma / Bazı Üretim     |
+-----------------+--------------------------+---------------------------+-----------------------------+
| OKL4            | Akademik / Araştırma     | Gömülü / Mobil            | Üretim (Bazı platformlar)   |
+-----------------+--------------------------+---------------------------+-----------------------------+
| Haiku           | Kısmen Mikro Kernel      | Genel Amaçlı (Masaüstü)   | Geliştirme Aşamasında       |
+-----------------+--------------------------+---------------------------+-----------------------------+

Genel olarak UNIX türevi sistemler tek parçalı çekirdekli biçimde tasarlanmaktadır. Aşağıda tek parçalı çekirdekli
tarafa yakın olan işletim sistemlerine örnekler veriyoruz:

Monolitik Çekirdekli Sistem Örnekleri
-------------------------------------

+------------------+----------------------------+------------------------------+
| Sistem           | Tür                        | Not                          |
+==================+============================+==============================+
| Linux            | Modüler Monolitik          | LKM desteği var              |
+------------------+----------------------------+------------------------------+
| FreeBSD          | Saf Monolitik              | POSIX uyumlu                 |
+------------------+----------------------------+------------------------------+
| NetBSD           | Saf Monolitik              | Taşınabilirlik odaklı        |
+------------------+----------------------------+------------------------------+
| OpenBSD          | Saf Monolitik              | Güvenlik odaklı              |
+------------------+----------------------------+------------------------------+
| Solaris          | Modüler Monolitik          | SVR4 tabanlı                 |
+------------------+----------------------------+------------------------------+
| AIX              | Modüler Monolitik          | IBM Power mimarisi           |
+------------------+----------------------------+------------------------------+
| Original Unix    | Saf Monolitik              | Tarihsel referans            |
+------------------+----------------------------+------------------------------+
| Windows 95/98    | Monolitik (Hibrit eğilimli)| Üretim dışı, tarihsel        |
+------------------+----------------------------+------------------------------+
| Klasik (Eski)    | Saf Monolitik              | Üretim dışı, tarihsel        |
| Mac OS           |                            |                              |
+------------------+----------------------------+------------------------------+

Tek parçalı çekirdekli tasarım genel olarak daha hızlı ve kırılgan, mikro çekirdekli tasarım ise genellikle daha
yavaş ve sağlam olma eğilimindedir. Aşağıdaki tabloda iki tasarım mimarisini avantaj ve dezavantaj bakımından
karşılaştırıyoruz:

Mimari Karşılaştırması
----------------------

+----------------------+---------------------------------------------------+------------------------------------------------+
| Kriter               | Monolitik Çekirdek                                | Mikro Çekirdek                                 |
+======================+===================================================+================================================+
| Performans           | [+] Sistem çağrıları doğrudan çekirdek uzayında   | [-] IPC (mesaj geçişi) ek yük getirir ve       |
|                      | işlenir; düşük gecikme                            | gecikmeyi artırır                              |
|                      | [+] Context switch maliyeti düşük                 | [-] Kullanıcı/çekirdek geçişi sık yaşanır      |
+----------------------+---------------------------------------------------+------------------------------------------------+
| Güvenilirlik /       | [-] Bir sürücü hatası tüm sistemi çökertebilir    | [+] Servisler kullanıcı uzayında izole çalışır |
| Kararlılık           | [-] Hata yayılımı (fault propagation)             | [+] Hatalı servis yeniden başlatılabilir,      |
|                      | engellenemez                                      | sistemi çökertmeden                            |
+----------------------+---------------------------------------------------+------------------------------------------------+
| Güvenlik             | [-] Çekirdek büyüdükçe güvenlik açığı yüzeyi      | [+] Minimal TCB (Trusted Computing Base)       |
|                      | genişler                                          | [+] Servisler birbirinden izole edilir         |
|                      | [-] Tüm kod aynı ayrıcalık seviyesinde            | [+] seL4 gibi formal doğrulama mümkün          |
+----------------------+---------------------------------------------------+------------------------------------------------+
| Modülerlik /         | [-] Yeni servis eklemek çekirdeği doğrudan        | [+] Servisler bağımsız geliştirilebilir        |
| Geliştirilebilirlik  | etkiler                                           | [+] Yeni sürücü/servis kullanıcı uzayına       |
|                      | [+] LKM ile kısmi modülerlik sağlanır             | eklenir, çekirdek değişmez                     |
+----------------------+---------------------------------------------------+------------------------------------------------+
| Donanım Erişimi      | [+] Donanıma yakın çalışma kolaylığı              | [-] Donanım sürücüleri kullanıcı uzayında      |
|                      | [+] DMA, interrupt handling doğrudan              | çalışır, donanım erişimi dolaylıdır            |
|                      | çekirdekten yönetilir                             | [-] Sürücü geliştirmesi daha karmaşık          |
+----------------------+---------------------------------------------------+------------------------------------------------+
| Bakım / Test         | [-] Çekirdek kod tabanı büyük ve karmaşık         | [+] Çekirdek küçük ve anlaşılır                |
|                      | [-] Hata ayıklama (debugging) güçtür              | [+] Her servis bağımsız test edilebilir        |
|                      | [+] Geniş topluluk ve araç desteği                | [-] IPC katmanı debug'ı karmaşık olabilir      |
+----------------------+---------------------------------------------------+------------------------------------------------+
| Uygun Kullanım Alanı | [+] Genel amaçlı sistemlerde olgunlaşmış          | [+] Gömülü, gerçek zamanlı ve güvenlik kritik  |
|                      | [+] Masaüstü, sunucu, HPC ortamları               | sistemler için idealdir                        |
|                      | [+] Linux, BSD aileleri kanıtlanmış               | [+] QNX, seL4 üretim ortamında başarılı        |
+----------------------+---------------------------------------------------+------------------------------------------------+