## Kişiselleştirilmiş Pardus İmajı Oluşturma

Resmi Pardus imajları kullanarak kişiselleştirilmiş imaj oluşturmak için gerekli paketler; squashfs-tools rsync genisoimage xorriso isolinux

İşlemler Debain 10 (Buster) tabanlı Pardus 19.3 işletim sisteminde uygulanmıştır.

Aşağıdaki komutları uygulamadan önce Debian tabanlı işletim sistemlerine biraz aşina olmak gerekir.


Paketleri güncelleyelim ve gerekli paketleri kuralım;

```bash
sudo apt update
sudo apt upgrade
sudo apt install squashfs-tools genisoimage xorriso rsync isolinux
```

Pardus resmi imajınını indirin, ev dizininin altında kalıp adında bir klasör oluşturun ve oraya Pardus isosunu taşıyalım

```bash
mkdir ~/kalıp
mv Pardus-19.3-SERVER-amd64 ~/kalıp
```

Sonraki adımda /mnt dizininde pardus dizini oluşturup oraya bağlayın ve ev dizininde canlı dizini oluşturup imaj içeriğini oraya çıkarın;

```bash
sudo mkdir /mnt/pardus
mkdir ~/canlı
sudo mount -o loop ~/kalıp/Pardus-19.3-SERVER-amd64 /mnt/pardus
sudo rsync --exclude=/live/filesystem.squashfs -a /mnt/pardus/ ~/canlı
```

filesystem.squashfs'yi sistem dizini oluşturarak oraya açalım;

```bash
sudo unsquashfs -d ~/sistem /mnt/pardus/live/filesystem.squashfs
```

chroot ile sistemde değişiklik yaparken internet bağlantısı için gerekli dosyaları kopyalayalım;

```bash
sudo cp /etc/resolv.conf ~/sistem/etc/
sudo cp /etc/hosts ~/sistem/etc/
```

Bağlanması gereken dizinleri (dizinlerin ne işe yaradıklarını tek tek araştırabilirsiniz) sistem dizinine bağlayıp chroot yapalım;

```bash
sudo mount --bind /dev/ ~/sistem/dev
sudo chroot ~/sistem
mount -t proc none /proc
mount -t sysfs none /sys
mount -t devpts none /dev/pts
```

Yerel ayarlarda sorun yaşamamak için bu komutları uygulayalım;

```bash
export HOME=/root
export LC_ALL=C
```

Artık bu aşamada Pardus imajında açık bir sistem gibi çalışabilirsiniz. Yaptığınız değişiklikler sistem adında oluşturduğumuz dizininde kaydedilecektir. Daha sonra filesystem.squashfs'yi sistem dizinine açma işleminin tam tersini yaparak kişiselleştirilmiş isonun dosya sistemini oluşturmuş olacağız. Bu aşamada yeni paketler kurabilir, kurulu paketleri görüntüleyebilir, paketleri kaldırabilir veya kurulu paketlerde konfigürasyonlar yapabilirsiniz. Örneğin kişiselleştirilmiş isonuzun içinde neofetch paketi kurulu gelsin istiyorsanız __*sudo apt update*__ komutuyla depoyu güncelleyip __*sudo apt install neofetch*__ komutunu uygulayabilirsiniz. 








Değişiklikleri uyguladıktan sonra temizliği yapalım;

```bash
apt clean
rm -rf /tmp/*
history -w
history -c
```

Bağladığımız dizinleri ayıralım ve chroot ortamından çıkalım;

```bash
umount /proc || umount -lf /proc
umount /sys
umount /dev/pts
cd
exit
sudo umount ~/sistem/dev
```

Dosya sistemini sıkıştırıp yeni filesystem.squashfs dosyamızı oluşturalım. İşlem sırasında bütün işlemciniz kullanılacaktır;

```bash
sudo mksquashfs ~/sistem ~/canlı/live/filesystem.squashfs
```

Dosya sistemi boyutunu güncelleştirelim;

```bash
sudo su
printf $(du -sx --block-size=1 ~/sistem | cut -f1) > ~/canlı/live/filesystem.size
exit
```

Yeni sistemin vmlinuz ve initrd dosyalarını taşıyalım;

```bash
sudo cp ~/sistem/boot/vmlinuz* ~/canlı/live/vmlinuz
sudo cp ~/sistem/boot/vmlinuz* ~/canlı/live/
sudo cp ~/sistem/boot/initrd* ~/canlı/live/initrd.img
sudo cp ~/sistem/boot/initrd* ~/canlı/live/
```

Eski md5sum.txt dosyasını kaldırıp yenisini oluşturalım;

```bash
cd ~/canlı
sudo rm md5sum.txt
find -type f -print0 | sudo xargs -0 md5sum | grep -v isolinux/boot.cat | sudo tee md5sum.txt
```


isolinux kullanarak gpt ve mbr disklerinde kurulumunu yapabileceğiniz Uefi ve Legacy destekli imajınızı aşağıdaki komutla oluşturalım. Aşağıdaki komut, parametreleriyle birlikte tek bir komuttur. Aşağıdaki komutu iso olarak çıkaracağımız dizinin bir üst dizininde çalıştırmamız gerekir.

```bash
xorriso -as mkisofs -isohybrid-mbr /usr/lib/ISOLINUX/isohdpfx.bin -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -eltorito-alt-boot  -e boot/grub/efi.img -no-emul-boot -isohybrid-gpt-basdat -isohybrid-apm-hfsplus -o "Pardus_Edited.iso"  ~/canlı
```

Oluşturduğunuz imajımızı dd komutuyla usb belleğe de yazdırabilirsiniz

```bash
sudo dd if=Yeni_Imjınızın_Adı.iso of=/dev/sda status="progress"
```

#### Kaynaklar;

SulinOS geliştiricisi Ali Rıza KESKİN

https://farukomercakmak.gitlab.io

https://basarmert.blogspot.com/

https://anadolupanteri.net/author/ramazan-altintop
