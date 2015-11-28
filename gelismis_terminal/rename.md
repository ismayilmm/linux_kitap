# rename

Aslında GNU/Linux sistemlerde dosyayı yeniden adlandırmak için bir komut bulunmaz. Bir dosyayı yeniden adlandırmak, onu taşımak demektir, dolayısıyla **mv** komutu bu iş için kullanılır.

rename ise Larry Wall tarafından yazılan bir Perl Scriptidir ve kompleks yeniden isimlendirme işleri için kullanılır. Regular Expression desteği sayesinde mv ile ancah bash scripti yazılarak çözülebilecek problemler rename ile çözülür. Modern sistemlerde genellikle yüklü gelir, ancak bulunmaması halinde paket yöneticilerinden rahatlıkla edinilebilir.

Örneğin elimizde rakamlardan oluşan jpg dosyaları olsun, ve bu dosyaların uzantılarına dokunmadan, başına sonuna kelime eklemek isteyelim.

```bash
eaydin@dixon ~/calisma/jpg $ ls
0145.jpg  0146.jpg  3.jpg  897653.jpg
eaydin@dixon ~/calisma/jpg $ rename -v 's/(\d*)\.jpg$/goruntu_$1_dosyasi\.jpg/' *.jpg
0145.jpg renamed as goruntu_0145_dosyasi.jpg
0146.jpg renamed as goruntu_0146_dosyasi.jpg
3.jpg renamed as goruntu_3_dosyasi.jpg
897653.jpg renamed as goruntu_897653_dosyasi.jpg
eaydin@dixon ~/calisma/jpg $ ls
goruntu_0145_dosyasi.jpg  goruntu_0146_dosyasi.jpg  goruntu_3_dosyasi.jpg  goruntu_897653_dosyasi.jpg
```

Eğer işlemi geri almak isteseydik, yani başında **kelime_** ve sonunda **_başkakelime** olup ortasında rakam bulunan jpg dosyalarını **rakam.jpg** şeklinde değiştirmek isteseydik,
