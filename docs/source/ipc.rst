=========================
Proseslerarası Haberleşme
=========================

Bu bölümde UNIX/Linux sistemlerinde *proseslerarası haberleşme (interprocess communication)* konusunu ele
alacağız. Proseslerarası haberleşme oldukça önemli ve ayrıntılı bir konudur.

Proseslerarası Haberleşme Nedir?
================================

İşletim sistemleri dünyasında bir prosesin başka bir prosese bilgi göndermesi ve başka bir prosesten bilgi
almasına *proseslerarası haberleşme (interprocess communication (IPC))* denilmektedir. Burada bilgi gönderip
almadan kastedilen şey bir grup byte'ın gönderilip alınmasıdır. Proseslerarası haberleşme önemli bir konudur.
Çünkü sayfa tabloları yoluyla birbirinden izole edilmiş proseslerin başka yöntemlerle birbirleriyle
haberleşmesi gerekebilmektedir. Bir program başka bir programa bir şeyler gönderebilir, o program da bunları
işleyebilir. Proseslerarası haberleşmenin en bariz örneği *client-server* sistemlerdir. Bu sistemlerde client
program server programa bir istek gönderir. Server program da bu isteği yerine getirip sonuçları client
programa iletir. Client-server tarzda haberleşmenin sağlanabilmesi için proseslerarası haberleşme denilen
mekanizmanın kullanılması gerekir. Benzer biçimde *dağıtık sistemlerde (distributed systems)* de esas olarak
proseslerarası haberleşme mekanizmaları kullanılmaktadır.

Proseslerarası haberleşme mekanizmaları iki bölümde ele alınıp incelenmektedir:

1) Aynı makinenin prosesleri arasında haberleşme

2) Farklı makinelerin prosesleri arasında haberleşme

Aynı makinenin prosesleri arasında haberleşme işletim sisteminin sağladığı özel yöntemlerle
gerçekleştirilmektedir. Farklı makinelerin prosesleri arasında haberleşme için ortak uyulması gereken bazı
kuralların belirlenmiş olması gerekir. Bu ortak kurallara *protokol (protocol)* denilmektedir. Farklı
makinelerin prosesleri arasında haberleşme için çeşitli protokol aileleri oluşturulmuştur. Ancak bunlardan en
yaygın kullanılanı *IP (Internet Protocol)* isimli protokol ailesidir. Biz kursumuzda önce aynı makinenin
prosesleri arasındaki haberleşme yöntemlerini inceleyeceğiz daha sonra başka bir bölümde farklı makinelerin
prosesleri arasındaki haberleşme yöntemleri üzerinde duracağız. O bölümde IP protokol ailesini ve TCP/UDP
haberleşmelerini ayrıntılı bir biçimde ele alacağız.

Aynı makinenin prosesleri arasındaki haberleşmeler işletim sistemine bağlı bir biçimde yani işletim sistemine
özgü bir biçimde yürütülmektedir. Ancak aynı makinenin prosesleri arasındaki haberleşmeler için işletim
sistemlerinde birbirine benzer yöntemler kullanılmaktadır. Örneğin *boru (pipe)* haberleşmesi yöntemi
UNIX/Linux sistemleriyle Windows sistemleri arasında benzer biçimde yürütülmektedir. *Paylaşılan bellek
alanları (shared memory)* denilen haberleşme yöntemi de yine UNIX/Linux sistemleriyle Windows sistemlerinde
benzer biçimde uygulanmaktadır. Ancak bazı işletim sistemlerinde o sisteme özgü özel yöntemler de
bulunabilmektedir. Tabii biz kursumuzda UNIX/Linux sistemlerindeki proseslerarası haberleşme yöntemleri
üzerinde duracağız.

Proseslerarası haberleşme mekanizmasının iki yönü vardır. Birincisi iki prosesin haberleşeceği bir ortamın
oluşturulmasıdır. İkincisi ise bu ortamın senkronize bir biçimde kullanılmasıdır. Yani proseslerarası
haberleşme bir senkronizasyon sağlanarak gerçekleştirilmelidir. Biz bu bölümde sürecin senkronizasyon kısmı
üzerinde durmayacağız. Thread senkronizasyonu thread'lerin anlatıldığı kısımda ele alınacaktır.

Boru (Pipe) Haberleşmesi
------------------------

En yaygın kullanılan proseslerarası haberleşme yöntemi *boru (pipe)* haberleşmesi denilen yöntemdir. Boru
haberleşmesi hem bir ortam sunarken hem de senkronizasyonu kendi içerisinde sağlamaktadır. Bu nedenle boru
haberleşmesi kolay kullanılabilen bir IPC yöntemidir. Boru haberleşmeleri *isimsiz boru haberleşmeleri* ve
*isimli boru haberleşmeleri* olmak üzere ikiye ayrılmaktadır. İsimsiz boru haberleşmelerine İngilizce *unnamed
pipe* ya da *anonymous pipe*, isimli boru haberleşmelerine ise *named pipe* ya da *fifo* denilmektedir. Bu
kursumuzda önce *isimsiz boru haberleşmesini*, sonra da *isimli boru haberleşmesini* anlatacağız.

Peki boru nedir? Boru aslında FIFO tarzında çalışan bir kuyruk sistemidir. Bu kuyruk sistemi çekirdek
tarafından çekirdek alanı içerisinde oluşturulmaktadır. Bir proses boruya yazma yapar, diğeri de borudan
yazılanları okur. Boruya yazılan byte'lar FIFO prensibine göre yazılırlar, böylece borudan okuyan taraf
olanları yazıldığı sırada okur. Örneğin boruya yazan taraf *ankara* yazısının karakterlerini boruya yazmış
olsun. Şimdi okuyan taraf önce 'a' byte'ını sonra da sırasıyla 'n', 'k', 'a', 'r' ve 'a' byte'larını borudan
okur.

.. image:: _static/ders43-boru.png
   :alt: Boru haberleşmesinde yazan prosesten okuyan prosese FIFO biçiminde veri akışı
   :align: center
   :width: 95%

UNIX/Linux sistemlerinde borular birer dosya gibi ele alınmaktadır. Dolayısıyla boruya yazma işlemi *write*
fonksiyonu ile borudan okuma işlemi ise *read* fonksiyonu ile yapılmaktadır.

Boruların belli bir uzunlukları vardır. Eskiden BSD ve Linux sistemlerinde boru uzunlukları 4096 byte (bir
sayfa) büyüklüğündeydi. Linux daha sonra default boru uzunluğunu 64K'ya (65536 byte'a (16 sayfaya))
yükseltmiştir.

UNIX/Linux sistemlerinde borular tek yönlüdür. Bu nedenle proseslerden biri boruya yazma yaparken diğeri
borudan okuma yapar. (Halbuki örneğin Windows sistemlerinde borular çift yönlüdür.) Borunun tek yönlü olması
demek iki prosesin de yazdıklarının aynı boruya yazılması demektir. P1 prosesi boruya yazma yapıp yine P1
prosesi borudan okuma yaparsa kendi yazdığını okur. Bunun da bir anlamı olmaz. Halbuki çift yönlü borularda
borudan hem okuma hem yazma yapılabilmektedir. Borudan okuma yapıldığında karşı tarafın yazdığı okunur. Ancak
UNIX/Linux sistemlerindeki yukarıda da belirttiğimiz gibi borular tek yönlüdür. Dolayısıyla haberleşmede bir
taraf yazma yaparken diğer taraf okuma yapmalıdır. Eğer borularla karşılıklı okuma yazma yapılmak isteniyorsa
iki boru kullanılmalıdır.
