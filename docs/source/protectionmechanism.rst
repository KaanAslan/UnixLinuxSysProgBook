
===============================
İşlemcilerin Koruma Mekanizması
===============================

Bu mini bölümde sistem programlama ile uğraşanların mutlaka bilmesi gereken *koruma mekanizması (protection mechanism)*
üzerinde duracağız. Koruma mekanizması ana hatlarıyla anlaşılmadan UNIX/Linux sistem programlamadaki bazı süreçlerin tam
olarak anlaşılması mümkün değildir.

Koruma Mekanizması Nedir?
=========================

Çok prosesli (multiprocessing) işletim sistemlerinin çalıştığı donanımlarda kullanılan mikroişlemcilerin
*koruma mekanizması (protection mechanism)* denilen bir özelliği vardır. Çok prosesli sistemlerde bütün
çalışan programlar o anda fiziksel bellekte (yani RAM'de) bir biçimde bulunmaktadır. Tabii işletim sisteminin 
kendisi de fiziksel RAM'de bulunur. Bir programın göstericiler yoluyla kendi bellek alanının dışına çıkarak başka bir prosesin bellek
alanına erişmesi mutlaka engellenmesi gereken bir durumdur. Çünkü eğer bu durum engellenmezse bir program
başka bir programın bellek alanını bozabilir. Bu bozulma da o programın hatalı çalışmasına ya da
çökmesine yol açabilir. Program başka bir programın bellek alanını bozmasa bile oradaki programlar üzerinde
casusluk faaliyetleri yürütebilir. Buna ek olarak bazı makine komutları tamamen sistemin çökmesine de yol
açabilmektedir. Bir programın bu makine komutlarını kullanması tüm sistemi çökertebileceği için bu durumun
da önüne geçilmesi gerekir. Örneğin Intel işlemcilerindeki 1 byte'lık ``CLI`` makine komutu o anda tüm sistemi
çökertebilmektedir. İşte işlemcilerin koruma mekanizması bu tür ihlallerin birinci elden işlemci tarafından
tespit edilip engellenmesini sağlamaktadır.

İşlemcilerin koruma mekanizmasının iki yönü vardır:

- Bellek Koruması
- Komut Koruması

Bellek koruması bir prosesin kendi bellek alanının dışına yaptığı erişimlerinin tespit edilmesine yönelik
mekanizmadır. Komut koruması ise sistemi çökertme potansiyeline sahip makine komutlarının kullanımının
engellenmesine yönelik mekanizmadır.

Bir prosesin bellek korumasını ve komut korumasını ihlal etmesi birinci elde işlemci tarafından tespit edilmektedir. 
İşlemci ihlali tespit eder ve işletim sistemine bildirir. İşletim sistemi de hemen her zaman programı sonlandırır.

Koruma Mekanizmasına Sahip İşlemciler
=====================================

Her türlü işlemci koruma mekanizmasına sahip değildir. Ancak güçlü işlemcilerde bu mekanizma bulunmaktadır. Örneğin
Intel'in 80386 ve sonrası işlemcileri, ARM'nin Cortex A serisi işlemcileri, Alpha işlemcileri, PowerPC işlemcileri,
Itanium işlemcileri bu mekanizmalara sahiptir. Mikrodenetleyiciler genel olarak küçük işlemciler oldukları için bu
mekanizmaya sahip değildir. Windows gibi Linux gibi macOS gibi işletim sistemleri bu mekanizmaya sahip olmayan
işlemcilerin bulundurduğu sistemlerde kullanılamamaktadır. Aşağıda koruma mekanizmasına sahip eski ve modern
işlemcilerin bazılarını bir liste halinde veriyoruz:

.. list-table::
   :widths: 30 34 36
   :header-rows: 1

   * - İşlemci
     - Ayrıcalık Düzeyleri
     - Bellek Koruması
   * - DEC VAX
     - 4 mod (Kernel-Exec-Super-User)
     - Sayfalama, dahili MMU
   * - Zilog Z8000
     - System / Normal
     - Harici MMU (Z8010), segment
   * - Intel iAPX 432
     - Yetenek (capability) tabanlı
     - Nesne tabanlı koruma
   * - Intel 80286
     - 4 halka (Ring 0-3)
     - Segment tabanlı, sayfalama yok
   * - Motorola 68010
     - Supervisor / User
     - Harici MMU (MC68451)
   * - Motorola 68020
     - Supervisor / User
     - Harici MMU (MC68851)
   * - Intel 80386 ve sonrası
     - 4 halka (Ring 0-3)
     - Segment + sayfalama, dahili MMU
   * - MIPS R2000 ve sonrası
     - Kernel / User (+ Supervisor)
     - TLB tabanlı sayfalama
   * - Motorola 68030 ve sonrası
     - Supervisor / User
     - Dahili MMU, sayfalama
   * - SPARC (v7/v8/v9)
     - Supervisor / User
     - Sayfalama (SRMMU / MMU)
   * - PowerPC
     - Supervisor / User (+ Hypervisor)
     - Sayfalama, BAT kayıtçıları
   * - DEC Alpha
     - Kernel-Exec-Super-User
     - Sayfalama, PALcode
   * - ARM ARM610 / ARM920T
     - Privileged / User
     - Dahili MMU, sayfalama
   * - Intel Itanium (IA-64)
     - 4 ayrıcalık düzeyi
     - Sayfalama, koruma anahtarları
   * - AMD64 / x86-64
     - 4 halka (pratikte 0 ve 3)
     - Sayfalama, NX biti
   * - ARM Cortex-M (ARMv7-M/v8-M)
     - Thread / Handler (priv/unpriv)
     - MPU (isteğe bağlı), sayfalama yok
   * - ARM Cortex-A (ARMv7-A/v8-A)
     - PL0-PL1 / EL0-EL3
     - MMU, sayfalama, iki aşamalı çeviri
   * - ARM Cortex-R
     - Privileged / User
     - MPU, sayfalama yok
   * - RISC-V (S-mode destekli)
     - M / S / U modları
     - Sv32/Sv39/Sv48 sayfalama, PMP

Bu tabloda bellek korumasının türünü de ayrı bir sütun olarak verdik. Buradaki tüm işlemcilerin komut koruması
zaten bulunmaktadır.

Çekirdek Modu ve Kullanıcı Modu
===============================

Koruma mekanizması kullanıcı modunda çalışan proseslerin thread'lerine uygulanmaktadır. Çekirdek içerisindeki 
kodların ve aygıt sürücü kodlarının bu koruma engeline takılmaması gerekir. Çekirdek belleğin her yerine 
erişebilmelidir. Çünkü programları bile fiziksel belleğe yükleyen çekirdektir. Aynı zamanda çekirdek sistemi 
çökertme potansiyelinde olan pek çok makine komutunu amaca uygun bir biçimde kullanmaktadır. Benzer biçimde 
aygıt sürücüler de işlevleri gereği bu tür makine komutlarını kullanabilmektedir. İşte çekirdek kodlarının 
ve aygıt sürücü kodlarının bir biçimde bu koruma mekanizmasından muaf tutulması gerekmektedir. Yani bu koruma 
mekanizması yalnızca kullanıcı programlarına uygulanmalıdır. Çekirdek kodları koruma engeline maruz kalmamalıdır.

İşlemcileri tasarlayanlar genellikle kod akışları için iki *ayrıcalık modu (privilege mode)* tanımlamaktadır: 
*çekirdek modu (kernel mode)* ve *kullanıcı modu (user mode)*. Çekirdek modu yerine *süpervizör modu (supervisor 
mode)* terimi de kullanılmaktadır. Eğer bir kod çekirdek modunda çalışıyorsa işlemci koruma mekanizmasını 
o kod için işletmez. Böylece o kod her şeyi yapabilir. Ancak eğer bir kod kullanıcı modunda çalışıyorsa işlemci 
o kod için koruma mekanizmasını işletmektedir. Normal programların hepsi kullanıcı modunda (user mode) çalışmaktadır. 
Ancak çekirdek kodları ve aygıt sürücüler (çekirdek modülleri) çekirdek modunda çalışırlar. Bazı işlemcilerde 
akşlar için ayrıcalık modları ikiden fazla da olabilmektedir. Örneğin Intel işlemcileri "halka (ring)" adı 
altında akışlar için dört ayrıcalık modunu desteklemektedir. Ancak Linux, Windows, macOS sistemleri bunlardan 
yalnızca ikisini kullanmaktadır. İşletim sistemlerinin kullandığı bu iki ayrıcalık modu zaten işlevsel olarak 
çekirdek moduna ve kullanıcı moduna karşılık gelmektedir. 

Bir programın ``sudo`` ile çalıştırılmasının (yani programın proses ID'sinin ``0`` olmasının) bu konuyla hiçbir
ilgisi yoktur. Proses ID'nin ``0`` olması yalnızca dosya erişimleri için ayrıcalık sağlamaktadır. Yoksa biz bir
programı ``sudo`` ile çalıştırsak bile o program yine kullanıcı modunda çalıştırılmaktadır ve koruma mekanizmasına 
tabi olmaktadır.

Peki biz kendi programımızı çekirdek modunda çalıştıramaz mıyız? Bu sorunun yanıtı genel olarak *hayır*
biçimindedir. Bunun tek yolu *aygıt sürücü* ya da *çekirdek modülü* denilen biçimde kodu yazmak ve onu
çekirdek alanına yerleştirmektir. Zaten aygıt sürücülerin en önemli özelliği onların çekirdek modunda
çalıştırılmasıdır. Tabii aygıt sürücüler ancak sistem yöneticisi tarafından bir parola eşliğinde (yani
``sudo`` ile) çekirdeğe yüklenebilmektedir.

Peki işlemci o anda çalıştırmakta olduğu kodun çekirdek modunda mı yoksa kullanıcı modunda mı olduğunu
nasıl anlamaktadır? İşte bu bilgi işlemcinin özel bir yazmacında tutulmaktadır. Zaman paylaşımlı çalışmada
thread'ler arası geçiş yapılırken bu yazmaç bilgileri de saklanıp geri yüklendiği için bazı thread'ler o
anda kullanıcı modundayken bazıları çekirdek modunda çalışıyor durumda olabilmektedir. Örneğin Intel
işlemcilerinde *CS (Code Segment Register)* yazmacının yüksek anlamlı ``2`` bitine *CPL (Current Privilege Level)*
denilmektedir. Bu iki bit prosesin modunu belirtmektedir. (Intel işlemcilerinde iki değil dört çalışma modu
olduğunu belirtmiştik.) 32 ARM işlemcilerinde bu bilgi *CPSR (Current Program Status Register)* isimli yazmacın, 
64-bit ARM işlemcilerinde ise *PSTATE isimli yazmacın* bir bitinde tutulmaktadır.

Akışın Kullanıcı Modundan Çekirdek Moduna Geçmesi ve Kullanıcı Moduna Geri Dönmesi
==================================================================================

Sistem fonksiyonları çekirdeğin içerisinde bulunmaktadır. Dolayısıyla bu fonksiyonlar özel makine
komutlarını kullanırlar ve bellekte her yere erişebilirler. Aksi takdirde bu fonksiyonların yazılabilmesi
mümkün değildir. Peki bizim programlarımız kullanıcı modunda çalıştığına göre biz bir sistem fonksiyonunu
çağırdığımızda ne olacaktır? İşte kullanıcı modunda çalışan bir proses bir sistem fonksiyonunu
çağırdığında proses otomatik olarak çekirdek moduna geçirilmektedir. Böylece sistem fonksiyonu yine
çekirdek modunda çalıştırılmaktadır. Sistem fonksiyonunun çalışması bittiğinde proses yine otomatik olarak
kullanıcı moduna dönmektedir. Örneğin Intel işlemcilerinde bu geçişi sağlayan mekanizmaya *kapı (gate)*
denilmektedir. Tabii kapı yerleştirmek ancak çekirdek modunda yapılabilecek bir işlemdir. Dolayısıyla
kullanıcı modundaki prosesler yalnızca "zaten belirlenmiş olan kodları çalıştırmak üzere" çekirdek moduna
geçebilmektedir.

Sistem fonksiyonlarını çağırmanın zamansal bir maliyeti vardır. Çünkü prosesin kullanıcı modundan çekirdek
moduna geçmesi ve birtakım gerekli kontrollerin çekirdek modunda yapılması zaman kaybına yol açmaktadır.
Örneğin:

.. code-block:: c

    read(fd, some_kernel_addr, 10)

Linux'ta ``read`` POSIX fonksiyonu doğrudan ``sys_read`` sistem fonksiyonunu çağırmaktadır. Eğer bu sistem
fonksiyonu ikinci parametreysiyle aldığı adresi hiç kontrol etmezse koruma mekanizmasından da muaf olduğu
için tuzağa düşecektir. Bu tür sistem fonksiyonları kendilerine verilen adreslerin o prosesin
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

Aşağıdaki şema, bir POSIX dosya fonksiyonunun kullanıcı modundan çekirdek moduna nasıl geçtiğini göstermek
için sadeleştirilmiştir:

.. figure:: _static/user-kernel-mode-switch.png
    :align: center
    :width: 80%
   