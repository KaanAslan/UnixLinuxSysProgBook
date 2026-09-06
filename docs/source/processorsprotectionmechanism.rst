
=================================================================
İşlemcilerin Koruma Mekanizması (Processors Protection Mechanism)
=================================================================

Koruma Mekanizmasına Giriş
==========================

Bu bölümde sistem programlama ile uğraşanların mutlaka bilmesi gereken *koruma mekanizması (protection mechanism)*
üzerinde duracağız. Koruma mekanizması ana hatlarıyla anlaşılmadan UNIX/Linux sistem programlamadaki bazı süreçlerin tam
olarak anlaşılması mümkün değildir.


İşlemci Koruma Mekanizması
==========================

Çok prosesli (multiprocessing) işletim sistemlerinin çalıştığı donanımlarda kullanılan mikroişlemcilerin
*koruma mekanizması (protection mechanism)* denilen bir özelliği vardır. Çok prosesli sistemlerde bütün
çalışan programlar o anda RAM'de bir biçimde bulunmaktadır. Tabii işletim sisteminin kendisi de RAM'de
bulunur. Bir programın göstericiler yoluyla kendi bellek alanının dışına çıkarak başka bir prosesin bellek
alanına erişmesi mutlaka engellenmesi gereken bir durumdur. Çünkü eğer bu durum engellenmezse bir program
başka bir programın bellek alanını bozabilir. Bu bozulma da o programın hatalı çalışmasına ya da
çökmesine yol açabilir. Program başka bir programın bellek alanını bozmasa bile oradaki programlar üzerinde
casusluk faaliyetleri yürütebilir. Buna ek olarak bazı makine komutları tamamen sistemin çökmesine de yol
açabilmektedir. Bir programın bu makine komutlarını kullanması tüm sistemi çökertebileceği için bu durumun
da önüne geçilmesi gerekir. Örneğin Intel işlemcilerindeki 1 byte'lık CLI makine komutu o anda tüm sistemi
çökertebilmektedir. İşte işlemcilerin koruma mekanizması bu tür ihlallerin birinci elden işlemci tarafından
tespit edilip engellenmesini sağlamaktadır.

İşlemcilerin koruma mekanizmasının iki yönü vardır:

- Bellek Koruması
- Komut Koruması

Bellek koruması bir prosesin kendi bellek alanının dışına yaptığı erişimlerinin tespit edilmesine yönelik
mekanizmadır. Komut koruması ise sistemi çökertme potansiyeline sahip makine komutlarının kullanımının
engellenmesine yönelik mekanizmadır.

Tabii her türlü mikroişlemci böyle bir mekanizmaya sahip değildir. Ancak güçlü işlemcilerde bu mekanizma
bulunmaktadır. Örneğin Intel'in 80386 ve sonrası işlemcileri, ARM'nin Cortex A serisi işlemcileri, Alpha
işlemcileri, PowerPC işlemcileri, Itanium işlemcileri bu mekanizmalara sahiptir. Mikrodenetleyiciler genel
olarak küçük işlemciler oldukları için bu mekanizmaya sahip değillerdir. Windows gibi Linux gibi macOS gibi
işletim sistemleri bu mekanizmaya sahip olmayan işlemcilerin bulundurduğu sistemlerde kullanılamamaktadır.

Bir prosesin bellek korumasını ve komut korumasını ihlal etmesi birinci elde işlemci tarafından tespit
edilmektedir. İşlemci ihlali tespit eder ve işletim sistemine bildirir. İşletim sistemi de hemen her zaman
programı sonlandırır.

Öte yandan çekirdek içerisindeki kodların ve aygıt sürücü kodlarının bu koruma engeline takılmaması
gerekir. Çekirdek belleğin her yerine erişebilmelidir. Çünkü programları bile belleğe yükleyen çekirdektir.
Aynı zamanda çekirdek sistemi çökertme potansiyelinde olan pek çok makine komutunu uygun bir biçimde
kullanmaktadır. Benzer biçimde aygıt sürücüler de mecburen bu tür makine komutlarını kullanabilmektedir.
İşte çekirdek kodlarının ve aygıt sürücü kodlarının bir biçimde bu koruma mekanizmasından muaf tutulması
gerekmektedir. Yani bu koruma mekanizması yalnızca kullanıcı programlarına uygulanmalıdır. Çekirdek
kodları koruma engeliyle karşılaşmamalıdır.

İşlemcileri tasarlayanlar genellikle prosesler için iki çalışma modu tanımlamaktadır: *çekirdek modu
(kernel mode)* ve *kullanıcı modu (user mode)*. Eğer bir kod çekirdek modunda çalışıyorsa işlemci koruma
mekanizmasını o kod için işletmez. Böylece o kod her şeyi yapabilir. Ancak eğer bir kod kullanıcı modunda
çalışıyorsa işlemci o kod için koruma mekanizmasını işletmektedir. Normal programların hepsi kullanıcı
modunda (user mode) çalışmaktadır. Ancak çekirdek kodları ve aygıt sürücüler (çekirdek modülleri) çekirdek
modunda çalışırlar.

Bir programın ``sudo`` ile çalıştırılmasının (yani programın proses ID'sinin 0 olmasının) bu konuyla hiçbir
ilgisi yoktur. Proses ID'nin 0 olması yalnızca dosya erişimleri için avantaj sağlamaktadır. Yoksa biz bir
programı ``sudo`` ile çalıştırsak bile o program yine kullanıcı modunda çalıştırılmaktadır ve koruma
mekanizmasına tabi olmaktadır.

Peki biz kendi programımızı çekirdek modunda çalıştıramaz mıyız? Bu sorunun yanıtı genel olarak *hayır*
biçimindedir. Bunun tek yolu *aygıt sürücü* ya da *çekirdek modülü* denilen biçimde kodu yazmak ve onu
çekirdek alanına yerleştirmekti. Zaten aygıt sürücülerin en önemli özelliği onların çekirdek modunda
çalıştırılmasıdır. Tabii aygıt sürücüler ancak sistem yöneticisi tarafından bir parola eşliğinde (yani
``sudo`` ile) çekirdeğe yüklenebilmektedir.

Sistem fonksiyonları çekirdeğin içerisinde bulunmaktadır. Dolayısıyla bu fonksiyonlar özel makine
komutlarını kullanırlar ve bellekte her yere erişebilirler. Aksi takdirde bu fonksiyonların yazılabilmesi
mümkün değildir. Peki bizim programlarımız kullanıcı modunda çalıştığına göre biz bir sistem fonksiyonunu
çağırdığımızda ne olacaktır? İşte kullanıcı modunda çalışan bir proses bir sistem fonksiyonunu
çağırdığında proses otomatik olarak çekirdek moduna geçirilmektedir. Böylece sistem fonksiyonu yine
çekirdek modunda çalıştırılmaktadır. Sistem fonksiyonunun çalışması bittiğinde proses yine otomatik olarak
kullanıcı moduna dönmektedir. Örneğin Intel işlemcilerinde bu geçişi sağlayan mekanizmaya *kapı (gate)*
denilmektedir. Tabii kapı yerleştirmek ancak çekirdek modunda yapılabilecek bir işlemdir. Dolayısıyla
kullanıcı modundaki prosesler yalnızca zaten belirlenmiş olan kodları çalıştırmak üzere çekirdek moduna
geçebilmektedir.

Sistem fonksiyonlarını çağırmanın zamansal bir maliyeti vardır. Çünkü prosesin kullanıcı modundan çekirdek
moduna geçmesi ve birtakım gerekli kontrollerin çekirdek modunda yapılması zaman kaybına yol açmaktadır.
Örneğin:

.. code-block:: c

    read(fd, (char *)0x123456, 10)

Linux'ta ``read`` POSIX fonksiyonu doğrudan ``sys_read`` sistem fonksiyonunu çağırmaktadır. Eğer bu sistem
fonksiyonu ikinci parametreyle verilen adresi hiç kontrol etmezse koruma mekanizmasından da muaf olduğu
için tuzağa düşecektir. İşte bu tür sistem fonksiyonları kendilerine verilen adreslerin o prosesin
kullanıcı alanı içerisinde olup olmadığını kontrol ederler. İşte çekirdek moduna geçildiğinde bunun gibi
çeşitli kontroller de yapılmaktadır.

O halde aslında bir proses yaşamının önemli bir kısmını kullanıcı modunda geçirirken bir kısmını da
çekirdek modunda geçirebilmektedir. Örneğin *time* isimli kabuk komutuyla biz prosesin ne kadar zamanı
çekirdek modunda ne kadar zamanı kullanıcı modunda geçirdiğini görebiliriz:

.. code-block:: console

    $ time ./sample

    real    0m0,189s
    user    0m0,185s
    sys     0m0,005s

Burada *sys* çekirdek modunu, *user* kullanıcı modunu ve *real* da toplam zamanı vermektedir.

Peki işlemci o anda çalıştırmakta olduğu kodun çekirdek modunda mı yoksa kullanıcı modunda mı olduğunu
nasıl anlamaktadır? İşte bu bilgi CPU'nun özel bir yazmacında tutulmaktadır. Zaman paylaşımlı çalışmada
thread'ler arası geçiş yapılırken bu yazmaç bilgileri de saklanıp geri yüklendiği için bazı thread'ler o
anda kullanıcı modundayken bazıları çekirdek modunda çalışıyor durumda olabilmektedir. Örneğin Intel
işlemcilerinde CS (Code Segment Register) yazmacının yüksek anlamlı 2 bitine CPL (Current Privilege Level)
denilmektedir. Bu iki bit prosesin modunu belirtmektedir. (Intel işlemcilerinde iki değil dört çalışma modu
vardır. Ancak pratikte işletim sistemleri bunların yalnızca ikisini kullanmaktadır.) Ancak Intel'in
dışındaki işlemcilerin hemen hepsi yalnızca iki modu kullanmaktadır.

Bazı işlemcilerde *kernel mode* terimi yerine *supervised mode* terimi de kullanılmaktadır. 32 bit ARM
işlemcilerinde bunun için CPSR (Current Program Status Register) isimli yazmacın, 64-bit ARM
işlemcilerinde ise PSTATE isimli yazmacın bir bitinde tutulmaktadır.

Aşağıdaki şema, bir POSIX dosya fonksiyonunun kullanıcı modundan çekirdek moduna nasıl geçtiğini göstermek
için sadeleştirilmiştir:

.. code-block:: text

    +----------------------+        +----------------------+        +------------------------+
    |  fopen               |  -->   |  open                |  -->   |  sys_open              |
    |  (kullanıcı modu)    |        |  (kullanıcı modu)    |        |  (çekirdek modu)       |
    +----------------------+        +----------------------+        +------------------------+
