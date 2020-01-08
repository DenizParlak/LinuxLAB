# Linux LAB

Bu simülasyon, her seviyeden Linux kullanıcısının kendisini test etmesi amacıyla hazırlanmıştır. "Şu şekilde de çözülebilirdi" dediğiniz cevaplarınız varsa gönderebilirsiniz.

LAB ortamına erişmek için Docker kullanmanız gerekmektedir. Komut:

```
docker run -ti denizparlak/lab
```

## Level 1

Göreviniz, tek find komutu kullanarak aşağıdaki şartları sağlayan dosya veya dosyaları bulmak:

– Sadece dosya tipinde olanlar

– sh uzantısına sahip olanlar

– chmod degeri 644

– Boyutu 900 byte’tan büyük, 5MB’tan küçük olanlar

## Çözüm

```find -type f -name “*.sh” -perm 644 -size +900c -size -5M```

Bu özelliklere sahip tek dosya: slackware.sh

## Level 2

Sistemde “hypnos” isimli bir kullanıcı bulunmaktadır. Göreviniz herhangi bir dosyayı editörle degistirmeden, yine tek komutla asagidaki degisiklikleri yapmak:

– Kullanıcı bilgisine “test” adında bir comment bölümü ekleyin.

– Ev dizinini /home/hypnos2 olarak güncelleyin.

– Kullanıcı grubunu “artemis” olarak degistirin.

– Kullanıcı ID’sini 1010 olarak degistirin.

– Kullanıcı kabugunu /bin/sh olarak degistirin.

## Çözüm

```
usermod -c “test” -u 1010 -d /home/hypnos2 -g artemis -s /bin/sh hypnos
```

Not: Bu soruda ikinci bir grup eklenmesi istenmiyor, doğrudan primary grup değiştiriliyor.

## Level 3

Her Salı gecesi 00:45'te /home/artemis/level1/dosyalar dizini içindeki klasörleri temizleyen bir cronjob oluşturun.

## Çözüm

```
45 0 * * 2 rm -rf /home/artemis/level1/dosyalar/*
```

veya

```
45 0 * * TUE rm -rf /home/artemis/level1/dosyalar/*
```

## Level 4

Bir metin dosyası içerisinde bulunan 4 segmentli, toplamda 16 karakterli bir numara dizisini, segment bazında ekrana tersten yazdıran bir bash script yazın.

Örneğin, numara 5829 8012 9370 2840 ise, sonuç 2840 9370 8012 5829 olmalıdır.

<img src="https://github.com/DenizParlak/LinuxLAB/blob/master/1.png" width="500">

## Çözüm

Yine birçok farklı yolu var. Örnek bir script:

```
#!/bin/bash

numbers=cc
declare -a arrV

arrV=(`cat "$numbers"`)

for (( i = 0; i < 1; i++))
do
  echo -n "${arrV[3]}"  "${arrV[2]}"  "${arrV[1]}"  "${arrV[0]}"
  echo -en "\n"
done
```

## Level 5

Bir metin dosyası içerisinde her bir satırda bulunan kullanıcı isimlerini sudoers grubuna otomatik olarak ekleyen bir bash script yazın.

## Çözüm

Bu soruda biraz kafa karışıklığı olmuş. Beklenen direkt metin dosyasından /etc/sudoers’e gerekli satırı eklemekti, çalışan sistem bir CentOS sürümü olduğu için, “sudoers” grubu denildiğinde aslında kullanılması gereken “wheel” olmalıydı, çözüm gönderenlerden kimse buna dikkat etmemiş 🙂

Direkt sudoers dosyasına eklemek için:

```
#!/bin/bash

users=b

while IFS= read -r line
do
  adduser $line
echo "$line  ALL=(ALL:ALL) ALL" >> /etc/sudoers

done < "$users"
```

“wheel” grubuna eklemek için:

```
#!/bin/bash

gr=$(< b)

for name in $gr
do
usermod -aG wheel $name
done
```

## Level 6

Editör kullanmadan, sadece komut satırından aşağıdaki SSH değişikliklerini yapın:

– root erişimi kapalı olmalı

– port 29 olarak değiştirilmeli

– MaxAuthTries 7, MaxSessions 9 olarak güncellenmeli

## Çözüm

Bu soruda da neredeyse herkes işin kolayına kaçıp “#” karakterini dahil etmeden çözüm göndermiş :) Beklenen, değişikliklerin aktif olarak kullanılabilecek hale getirilmesiydi ki bunu benim belirtmem gerekiyordu sanırım.

Soruda halledilmesi gereken ikinci problem, yaptığınız değişiklikler dosyadaki tüm match olan satırları değiştireceği için (e.g Port 29 – Port 29), istenilen değişikliklerin sadece birinci veya ikinci eşleşmede yapılması gerekliydi. Bu durum MaxAuthTries ve MaxSessions için geçerli değil.

```
sed -i '0,/.*PermitRootLogin.*/ s/.*PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
sed -i '0,/.*Port.*/ s/.*Port.*/Port 30/' /etc/ssh/sshd_config
sed -i "s/.*MaxAuthTries.*/MaxAuthTries 7/g" /etc/ssh/sshd_config
sed -i "s/.*MaxSessions.*/MaxSessions 9/g" /etc/ssh/sshd_config
```

<img src="https://github.com/DenizParlak/LinuxLAB/blob/master/2.png" width="500">

## Level 7

flume isimli dosyadaki metin üzerinde vi editörü ile aşağıdaki değişiklikleri yapmanız beklenmektedir:

– “and” kelimesini “&” karakteri ile değiştirin.

– Sadece 10. satırda bulunan “made” kelimelerini “mad3” olarak değiştirin.

– 5. ve 20. satırlar arasındaki “made” kelimelerini silin.

not: Tüm işlemler için command mode’u kullanmanız gerekmektedir.

## Çözüm

Yine birden farklı yolla çözülebilirdi ki editör kullanmadan sed ile çözüm gönderenler olmuş, notta da belirtildiği gibi işlemler “command mode” ile yapılmalıydı. Sırasıyla:

```
:%s/and/\&/g

:10,10s/made/mad3/g

:5,20s/made//g
```

## Level 8

nevermind isimli dosya üzerinde aşağıdaki işlemleri yapmanız beklenmektedir:

– Toplam karakter ve satır sayısını STDOUT kullanarak a1 dosyasına yazdırın.

– Tek komutla tüm “take” kelimelerini dosyadan silin.

– Tüm karakterler arasında birer boşluk olacak şekilde yine tek komutta dosyayı güncelleyin.

## Çözüm

```
wc -c < nevermind >> a1 && wc -l < nevermind >> a1

sed -i ‘s/take//g’ nevermind
```

Son seçenekte istenen anlaşılmamış 🙂 “Tüm” karakterler arasında “birer” boşluk diyor arkadaşlar:

<img src="https://github.com/DenizParlak/LinuxLAB/blob/master/3.png" width="500">

```
sed -e ‘s/(.)/\1 /g’ nevermind
```

Tekrardan teşekkürler, devamı gelecek.
