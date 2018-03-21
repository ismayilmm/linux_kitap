## Standart Hata ve File Descriptor

Şimdiye kadar standart girdi ve çıktı yönlendirmelerini, UNIX pipeline dahilinde programların birbirleriyle iletişim sağlayabilmesi için nasıl kullandığımızı gördük.

Aslında UNIX üzerinde programların üç temel veri akış biçimi vardır. Standart girdi, standart çıktı ve standart hata. İngilizceleri _Standard Input_, _Standard Output_ ve _Standard Error_ olarak geçer. Bu yüzden bazen kısaltmalarını _stdin_, _stdout_, _stderr_ olarak görebilirsiniz.

Standart girdi ve standart çıktının varoluş amaçlarını biraz önceki örnelerde irdeledik. Standart hata ise, program çalıştırıldığında, programın hatalarını yönlendireceği noktayı belirtir. Normal şartlar altında bir program sonuçlarını da \(standart çıktı\) hatalarını da \(standart hata\) ekranımıza yönlendirir. Böylece program hata verdiyse, çıktıları arasında görürüz. Ancak bu her zaman istediğimiz bir şey olmayabilir. Özellikle çok fazla çıktı veren programlarda gerçekleşen hataların ayıklanması zorlaşabilir. Ayrıca, programların standart çıktılarını başka program veya dosyalara yönlendirdiğimizde, hataların da burada yer almasını istemeyebiliriz. Bu tip durumların önüne geçebilmek için, standart çıktıdan bağımsız olan bir çıktı biçimi olarak standart hata tanımlanmıştır. Bu sayede örneğin bir programın standart çıktısı pipe ile bir başka programa girdi olarak sunulurken, programın çalışması sırasında oluşacak hatalar kullanıcının terminal ekranında görülebilir.

Standart hata kavramı, UNIX'in 6. versiyonunun ardından Dennis Ritchie tarafından geliştirilmiştir. Bu yöntem ile birbirine standart girdi-çıktı yönlendiren programların standart hatalarının terminale yazdırılması durumunda, hangi hata mesajının hangi programdan geldiğinin bilinmemesi problemiyle de karşılaşılmıştır. Douglas McIlroy, bu problemin farkında olduklarını ama asla tamamen çözülmediğini, _UNIX Programmer's Manual_'ın 3. versiyonunda belirtir.

> All programs placed diagnostics on the standard output. This had always caused trouble when the output was redirected into a file, but became intolerable when the output was sent to an unsuspecting process. Nevertheless, unwilling to violate the simplicity of the standard-input-standard-output model, people tolerated this state of affairs through v6. Shortly thereafter Dennis Ritchie cut the Gordian knot by introducing the standard error file. That was not quite enough. With pipelines diagnostics could come from any of several programs running simultaneously. Diagnostics needed to identify themselves. Thus began a never quite finished pacification campaign: a few recalcitrant diagnostics still remain anonymous or appear on the standard output.

Standart hata'nın kullanımı için önce, _file descriptor_ konsepti hakkında kısaca fikir sahibi olmamızda fayda var.

## File Descriptor

UNIX üzerinde aslında her şey bir dosyadır. Klavyeniz, seri port cihazınız, ethernet cihazınız, yazıcınız, metin dosyanız, ekranınız... Tamamını UNIX dosya sistemi birer dosya olarak tanımlar. Programlarınız bu dosyalara erişmek istediklerinde, işletim sistemi tarafından erişim izniniz olup olmadığı kontrol edilir, ardından sistem programa "negatif olmayan bir tam sayı" olarak dosyayı sunar. Kısacası program, aslında dosyanın ismiyle, yoluyla neredeyse hiç ilgilenmez, programın etkileşim kurduğu "şey" bir tam sayıdır, bu tam sayı da işletim sistemi tarafından dosya yoluna tercüme edilir. Bu tam sayılar, program ile işletim sistemi arasındaki dosya tanımını yaptığı için, _file descriptor_ ismini alır. Erişilmiş bir dosya hakkındaki bütün bilgi işletim sisteminin sorumluluğundadır, yazılımlar bu dosyaları sadece birer file descriptor olarak tanırlar. Standart girdi, standart çıktı ve standart hata, ayrı file descriptor'lar ile temsil edilir.

Daha önce incelediğimiz `stdio.h` için bu gösterimler stdin, stdout, stderr iken, bunlara karşılık gelen tam sayılar da genellikle bash gibi kabuklarda kullanılır.

| İsim | Sayısal Değer | &lt;stdio.h&gt; Kullanımı |
| :--- | :--- | :--- |
| Standart Girdi | 0 | stdin |
| Standart Çıktı | 1 | stdout |
| Standart Hata | 2 | stderr |

Daha önce gördüğümüz **&lt; ** ve **&gt;** gösterimleri de aslında bunu doğrular niteliktedir. Programlar girdinin nereden geldiğini ve çıktının nereye gideceğini bilmezler, onlar için sadece 0, 1 ve 2 \(veya stdin, stdout ve stderr\) vardır. Kabuk aracılığıyla **&lt; ** ve **&gt;** gösterimlerini kullanarak aslında işletim sistemi hangi programın nereden veri alıp nereye ileteceğini kontrol eder.

Yukarıdaki tablodan görüleceği gibi, aslında standart çıktının file descriptor'ının sayısal karşılığı 1'dir. Yani, aşağıdaki iki örnek, aynı anlama gelmektedir. İlk başta, daha önce uyguladığımız örneklerde olduğu gibi, file descriptor kullanmadan oluşan sonuca bakalım.

```
eaydin@eaydin-vt ~/devel/lower $ cat karakterler > cikti1
eaydin@eaydin-vt ~/devel/lower $ cat cikti1 
AbCdE
```

Aşağıda ise, file descriptor ile aynı sonucun elde edildiğini görebiliyoruz.

```
eaydin@eaydin-vt ~/devel/lower $ cat karakterler 1> cikti2
eaydin@eaydin-vt ~/devel/lower $ cat cikti2
AbCdE
```

Yönlendirmelerde standart çıktı çok sık kullanıldığı için, `1>` kullanımı olmadan da standart çıktı yönlendirmesi gerçekleştirilir.

Şimdi benzer işlemi standart hata yönlendirmesinde kullanalım.

cat programı, kendisine parametre olarak kaç dosya verilirse, tamamını peş peşe eklemekle görevlidir, ismi de zaten buradan gelir, _con**cat**enate_ sözcüğünün kısaltılmışıdır. Aşağıdaki kullanım açıklayıcı olacaktır.

```
eaydin@eaydin-vt ~/devel/lower $ echo Bu bir cümle > cumleler
eaydin@eaydin-vt ~/devel/lower $ cat cumleler 
Bu bir cümle
eaydin@eaydin-vt ~/devel/lower $ cat karakterler cumleler 
AbCdE
Bu bir cümle
```

Eğer parametre olarak verdiğimiz dosyalardan birisi yoksa \(veya okunamıyorsa\), hata verir.

```
eaydin@eaydin-vt ~/devel/lower $ cat paragraf
cat: paragraf: No such file or directory
```

Aşağıda, iki dosyayı birleştirmesini istiyoruz.

```
eaydin@eaydin-vt ~/devel/lower $ cat karakterler paragraf > deneme
cat: paragraf: No such file or directory
eaydin@eaydin-vt ~/devel/lower $ cat deneme
AbCdE
```

Ancak gördüğünüz gibi, dosyalardan birisi \(`paragraf`\) olmadığı için program hata verdi. Yine de `deneme` dosyası oluşturuldu ve içinde sadece `karakterler` dosyasının içeriği yer alıyor. Eğer çok fazla dosyayı birleştiriyor olsaydık, veya bu işlemi bir script'in içerisinde kullanıyor olsaydık, veya pipe ile birçok işlemi birleştiriyor olsaydık, bu işlemin hatalarını terminal ekranına yazdırmak yerine bir dosyaya yönlendirmesini tercih edebilirdik. Bunun için file descriptor kullanımı gerekir ve yukarıdaki tablodan anlaşılacağı gibi `2>` notasyonuyla bu işlem gerçekleştirilir.

```
eaydin@eaydin-vt ~/devel/lower $ cat karakterler paragraf > deneme 2> hatalar
eaydin@eaydin-vt ~/devel/lower $ cat deneme 
AbCdE
eaydin@eaydin-vt ~/devel/lower $ cat hatalar
cat: paragraf: No such file or directory
```

Burada önce standart çıktıyı `deneme` dosyasına yönlendirdiğimizi, standart hatayı ise `hatalar` dosyasına yönlendirdiğimizi görebilirsiniz. Yaptığımız işlemin sırasının bir önemi yok. Yani önce standart hatayı yönlendirip, sonra standart çıktıyı yönlendirebilirdik. Genellikle bu yapılmaz \(1 ve 2 sırasını içgüdüsel olarak koruruz\) ancak yapılmasında bir mahsur bulunmaz.

```
eaydin@eaydin-vt ~/devel/lower $ cat karakterler paragraf 2> hatalar > deneme
eaydin@eaydin-vt ~/devel/lower $ cat hatalar
cat: paragraf: No such file or directory
eaydin@eaydin-vt ~/devel/lower $ cat deneme
AbCdE
```

Bütün bu işlemlerde, standart çıktı yönlendirmesi için **1&gt;** notasyonunu da kullanabilirdik. Yine, alışkanlık gereği pek kullanılmaz sadece. Yani aşağıdaki işlem ile yukarıdaki aynı işe yarayacaktır.

```
eaydin@eaydin-vt ~/devel/lower $ cat karakterler paragraf 2> hatalar 1> deneme
```

### Open File Descriptor Limiti

File descriptor'ların UNIX sistemimiz üzerinde aslında çekirdek \(kernel\) tarafından idare edildiğini öğrendik. Yani bir program standart çıktısının nereye yazıldığını kendisi bilmiyor, ancak işletim sistemi her programın standart çıktısının \(ve standart girdisinin ve standart hatasının\) nereye işaret ettiğini, dolayısıyla her programın file descriptor'larının nereye karşılık geldiğini biliyor. Bu durum UNIX çekirdeğinin pek çok program için pek çok dosya işaretini _aklında tutmasına_ sebep olur. Buradaki "aklında tutması" aslında sistemin bunu _RAM'de tutması_ anlamına gelir. Bu yüzden sistemde file descriptor'ların bir limiti bulunur. Buna "o anda sistemin aklında tuttuğu file descriptor'lar" anlamına gelen **open file descriptor limit** denilir.

Daha önce `komut1 | komut2 | komut3` şeklinde bir pipeline oluştururken aslında _-neredeyse sonsuza kadar-_ bu diziyi uzatabileceğinizi söylemiştik. Buradaki "neredeyse" kısmı da bu limitten kaynaklanır. Aslında sisteminizin open file descriptor limiti kadar uzun bir dizi oluşturabilirsiniz, çünkü buradaki her program için farklı file descriptor'lar tanımlanmaktadır. İşletim sistemi bunların ne kadarını aynı anda aklında tutabilirse, o kadar komutu pipeline içerisinde kullanabilirsiniz demektir. Yine de bu limit, sunucular üzerinde pratik kullanımlarda fark etmeyeceğiniz kadar yüksektir. Genellikle gömülü sistemlerde \(kaynakların ekonomik sebeplerle düşük tutulduğu sistemlerde\) problem teşkil etmeye başlar.

Sisteminiz üzerinde bu limitler farklı biçimlerde temsil edilir. İşletim sisteminizin tamamının open file descriptor limitini öğrenmek için `/proc/sys/fs/file-max` dosyasının içeriğine bakmanız yeterli olacaktır. Örneğin:

```
[root@emre ~]# cat /proc/sys/fs/file-max
386774
```

Burada incelediğimiz sistemin _aynı anda_ 386774 tane file descriptor'ın açık olmasını desteklediğini görüyoruz. Yani UNIX pipeline'ında peş peşe çalıştırdığımız komutlar maksimum bu sayıya ulaşacak kadar file descriptor oluşturabilirler, ve tabii bu sırada çalışan programları da \(network servislerinin sağlanması, init programı, varsa çalışan veritabanları, başka yazılımlar vb.\) göz önünde bulundurmak gerekir. Hiç pipeline oluşturmasak bile, aslında çalışan programların da toplam bu kadar file descriptor oluşturabileceği anlamına gelir.

Ancak bu durum, tek bir programın \(veya kullanıcının\) 386774 limitinin çok çok büyük bir kısmını, örneğin 386000 tanesini işgal etmesine sebep olabilir. Bu tip durumların önüne geçmek için modern işletim sistemlerinde program başına open file descriptor limiti bulunmaktadır. Bunu öğrenmek için aşağıdaki komutu çalıştırabilirsiniz:

```
[root@emre ~]# ulimit -n
1024
```

Buradan görüleceği üzere, aslında bir program çalıştırıldığında, kendisine işletim sisteminin çekirdeği tarafından 1024 tane file descriptor oluşturma hakkı tanınır.

_"Ama önceki bölümlerde programların stdin, stdout ve stderr şeklinde 3 tane file descriptor'ı olduğunu söylemiştik? Tek program neden 1024 tane file descriptor'a ihtiyaç duysun ki?"_

Doğru, ve bu sorunun cevabını ilerleyen bölümlerde, file descriptor'ların doğasını daha derinlemesine irdelediğimizde alacağız. Şimdilik bu soruyu bir kenara bırakalım.

Sistemimizdeki open file descriptor limiti, program başına olsun veya olmasın, işletim sistemimizin limitlerine ve RAM'ine bağlı olduğu için, hali hazırda çekirdeğin aklında tuttuğu file descriptor sayısını görmek isteyebiliriz. Bunun için aşağıdaki komutu çalıştırabiliriz:

```
[root@emre ~]# cat /proc/sys/fs/file-nr
736    0    386774
```

Burada yine biraz önceki sayı olan 386774'ü, yani üst limiti görüyoruz. İlk baştaki 736 ise aslında sistemin şu anda aklında tuttuğu file descriptor sayısıdır. Dolayısıyla bu sistem üzerinde `386774-736=386038` tane daha file desciptor açabiliriz, ancak bunları programlara \(process'lere\) yaymak gerekecektir. Ortadaki 0 sayısı ise, sistem tarafından rezerve edilmiş \(_allocated_\) ancak kullanılmayan file descriptor'ların sayısını gösteriyor. Yani bu örnekte sistem "rezerve ettiği" bütün file descriptorları \(736 tane\) kullanmış.

Bu limitlerin nasıl düzenleneceğini, program başına neden limitler olduğunu biraz daha ilerleyen bölümlerde irdeleyeceğiz. Şimdi programların üç temel file descriptor'ına geri dönelim.

### Standart Hatanın Standart Çıktıya Yönlendirilmesi

Her ne kadar standart hata yönlendirmesi, standart çıktı ile aynı noktaya yazılmasını istemediğimizden dolayı ortaya çıkmış olsa da, bazı durumlarda hata ve çıktıyı aynı yere yazmak isteyebiliriz. Bu gibi durumlar için, file descriptor kullanımında farklı bir gösterim kullanılır.

```
eaydin@eaydin-vt ~/devel/lower $ cat karakterler cumle > sonuc 2>&1
eaydin@eaydin-vt ~/devel/lower $ cat sonuc
AbCdE
cat: cumle: No such file or directory
```

Yukarıdaki örnekte, yine `cat` programı ile `karakterler` ve `cumle` dosyalarını okuyup birleştirmek istedik. Birleşmiş çıktıyı da `sonuc` isminde bir dosyaya yazmak istedik. Ancak `cumle` diye bir dosyamız bulunmadığı için, programın hata vermesini bekleriz. Fakat programı çalıştırırken, standart hatayı da, standart çıktıya yazmasını söyledik. Standart çıktımız `sonuc` dosyası olduğu için, hatalarımızı ekranda görmek yerine, `sonuc` dosyasına görmeyi bekleriz. Gerçekten de `sonuc` dosyasının içeriğine baktığımızda hem `karakterler` dosyasının içeriğini, hem de `cumle` dosyası bulunamadığı için `cat` programınım verdiği hatayı görüyoruz.

Burada standart hatayı, standart çıktıya yönlendirmek için kullanılan notasyon biraz farklı görünebilir: `2>&1`

Bu gösterimin ne anlama geldiği, aslında yine Brian Kernighan ve Dennis Ritchie'nin The C Programming Language kitabının 8. bölümünde yatıyor:

> A file descriptor is analogous to the file pointer used by the standard library \[...\]

Yani, bir _file descriptor_, aslında C programlama dilinde standart kütüphanenin kullandığı _file pointer_'lar ile aynı işi yapıyor. C'de pointer gösterimi **&** işaret ile yapılmakta. Bunu, daha önce 1 olarak tanımlanmış bir değerin adresi olarak düşünebilirsiniz. Yukarıdaki bash satırında da, `cat` programının standart çıktısı için sonuc tanımlandı. Dolayısıyla 1 \(stdout\) değeri için bir adres belirtmiş olduk. Daha sonra 2 \(stderr\) değeri için adres belirtirken de "daha önce 1 için tanımladığım **adresi** kullan" demiş oluyoruz. Burada **adres** sözcüğüne karşılık gelmesi için de, C'deki pointerlardaki gösterim gibi **&** işareti kullanılmaktadır.

Burada **&** işaretini, bir programı arka planda çalıştırmak için sonuna koyduğumuzdaki kullanımıyla karıştırmamakta fayda var. Eğer ardından 1 veya 2 geliyorsa, doğrudan file desriptor için pointer görevi görmektedir.

Yukarıdaki işlemi de yine tersi olarak yapabilirdik. Yani önce standart hata yönlendirmesi yapıp, daha sonra standart çıktının yönleneceği yerin, standart hatanın adresi olmasını da söyleyebilirdik.

```
eaydin@eaydin-vt ~/devel/lower $ cat karakterler cumle 2> hatalisonuc 1>&2
eaydin@eaydin-vt ~/devel/lower $ cat hatalisonuc 
AbCdE
cat: cumle: No such file or directory
```

### Çıktı Yönlendirme Örneği

Genellikle standart hatayı ve standart çıktıyı birlikte yönlendirme işini, kalabalık çıktı sunan bir programın arka planda çalışmasında kullanırız. Örneğin checkmate.py isimli geliştirdiğimiz bir Python programı, belirli dizinlerdeki dosyaları tarayıp içlerinde bozuk dosya olup olmadığını kontrol etmektedir. Bu programın çıktısı "dosyaları kontrol ettim, problem yok" veya "dosyaları kontrol ettim, problem var" şeklinde olabilir. Kontrol işlemini bitirince email ile uyarı göndermektedir. Bu programın oluşabilecek hatası ise, kontrol edeceği dizinlere erişememesi durumunda gerçekleşebilir. Öyleyse aşağıdaki gibi programı çalıştırırsak:

```
root@ubuntu:~# ./checkmate.py >> /var/log/checkmate.log 2>&1
```

Program çalışmaya başlayacak, ancak hem "kontrol ettim problem var/problem yok" gibi uyarılarını, hem de programın akışında yaşayacağı problemleri aynı dosyaya, `/var/log/checkmate.log` dosyasına yönlendirecektir. Ayrıca bu dosyada mevcut log satırlarını silmeyecektir, çünkü **&gt;&gt;** işaretiyle satırları sonuna eklemek istediğimizi belirttik.

Terminalimizde yeni bir işlem yapmak için, programın sonlanmasını beklememiz gerekir. Eğer bunu istemeseydik, sonuna bir **&** işareti daha eklememiz gerekecekti.

```
root@ubuntu:~# ./checkmate.py >> /var/log/checkmate.log 2>&1 &
```

Buradaki iki **&** işaretinin farklı görevler gördüğünün daha önce altını çizmiştik. `2>&1` gösterimindeki **&** işaret, standart hatanın, standart çıktının adresine yönlenmesini sağlamaktadır. Ardından gelen **&** işareti ise bütün işlemin arka planda yürütülmesini söyler.

Bazı durumlarda programlar arka planda çalışırken, standart hataları doğru yere yönlendirilmez. Aşağıdaki gibi bir kullanım buna karşılık gelir:

```
root@ubuntu:~# ./checkmate.py >> /var/log/checkmate.log &
```

Bu durumda, `checkmate.py` programı arka planda işini yapacak ve "kontrol ettim hata var/yok" bilgilerini yine doğru log dosyasına yazacaktır. Bu işlemler gerçekleştirilirken biz de aynı terminal ekranında standart işlerimize devam edebilir oluruz. Ancak eğer bu sırada `checkmate.py` programı bir hatayla karşılaşırsa, bunu terminal ekranına yazdıracaktır. Bazen terminal üzerinde çalışırken durduk yere bu tip mesajlarla karşılaşabilirsiniz. Bu durumlar çoğunlukla arka planda çalışan bir işlemin standart hatasının doğru yönlendirilmemesinde kaynaklanır.

Genellikle standart çıktıyı ve standart hatayı aynı yere yönlendirme işlemini `/dev/null` dosyası ile görebilirsiniz. İleride göreceğimiz özel dosyalardan birisi olan `/dev/null`, içine yazılan her şeyi silen özel bir dosyadır. Böylece, örneğin `checkmate.py` programımızın çalışmasını istiyorsak, ancak oluşturacağı çıktılarla hiç ilgilenmiyorsak, bütün çıktılarını `/dev/null` dosyasına yönlendirip, bu çıktıların sistemde tutulmamasını sağlayabiliriz.

```
root@ubuntu:~# ./checkmate.py > /dev/null/ 2>&1
```

Bu tip kullanımı, en çok \(yine ileride göreceğimiz\) zamanlanmış görevlerde görürüz. `crontab` içine yazılan satırların çoğu, eğer loglanmasını istemediğimiz işlemler yapılıyorsa bu şekilde yazılır.
