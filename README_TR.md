# Panter-OS: Custom Kali Linux Live Builder

Siber güvenlik laboratuvarı ihtiyaçları doğrultusunda özelleştirilmiş, bağımsız ve taşınabilir bir Kali Linux Live ISO oluşturmak için hazırlanmış bir otomasyon projesi.

**Bu çalışma, Linux dağıtımı özelleştirme ve yeniden derleme süreçlerini öğrenmek amacıyla hazırlanmıştır.**

## 🎯 Projenin Amacı

Bu proje Kali Linux tabanlı özelleştirilmiş bir Live ISO üretir. Hostname, MOTD, terminal görünümü, duvar kağıdı ve paket seçimi build sürecinde otomatik olarak uygulanır.

Tüm özelleştirmeler live-build yapılandırmaları ve hook scriptleri ile otomatik olarak uygulanmaktadır.

---

### 🌍 Farklı İşletim Sistemlerinde Derleme
Bu proje, çalışmak için Debian/Kali tabanlı bir Linux ortamına ve `apt` paket yöneticisine ihtiyaç duyar. 

Eğer Windows veya macOS kullanıyorsanız, derleme işlemlerini VirtualBox veya VMware üzerinden kurduğunuz bir Linux (Debian/Ubuntu/Kali) sanal makinesinde yapmalısınız. Doğrudan Windows veya macOS üzerinde çalışmaz.

---

## 💻 Sistem Gereksinimleri ve Ortam Önerisi

Derleme sürecinde disk ve bellek darboğazları (`xorriso` ve `tmpfs` hataları) yaşamamak için aşağıdaki gereksinimlerin karşılanması **kritiktir**.

* **Önerilen Geliştirme Ortamı:** Derleme ve test işlemlerinin ana makine (host) yerine **VirtualBox** üzerinde oluşturulmuş temiz bir sanal makinede yapılması tavsiye edilir.
* **İşletim Sistemi:** Debian, Ubuntu veya Kali Linux (Sanal makine üzerinde).
* **Boş Disk Alanı:** En az **30 GB** (Derleme esnasında oluşan geçici chroot ve rootfs dosyaları için gereklidir).
* **RAM:** Geliştirme ortamı için en az **4 GB** (İdeal olarak 8 GB).

---

## Özellikler/Proje Çıktısı

- Özel hostname (cyber-node-01)
- Özel MOTD (PANTER-OS)
- Özel terminal görünümü ([SEC-LAB])
- Özel duvar kağıdı
- XFCE tabanlı masaüstü
- Live ISO desteği
- VirtualBox uyumluluğu
- Ön yüklü laboratuvar araçları (nmap, wireshark, tmux)

---


## 🚀 Adım Adım Kurulum ve Derleme Rehberi

Aşağıdaki adımları sırasıyla uygulayarak kendi Panter-OS ISO dosyanızı derleyebilirsiniz.

### Adım 1: Temel Kütüphanelerin Kurulumu
Derleme motoru ve gerekli bağımlılıkları sisteme kurun:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl git live-build cdebootstrap xorriso
```

Kontrol
```bash
dpkg -l | grep -E "live-build|xorriso|cdebootstrap"
```

### Adım 2: Proje İskeletinin Klonlanması
Kali'nin resmi derleme altyapısını indirin ve özelleştirme klasörlerini oluşturun:

```bash
cd ~
git clone https://gitlab.com/kalilinux/build-scripts/live-build-config.git panter-os
cd panter-os

mkdir -p kali-config/common/includes.chroot/etc/skel/
mkdir -p kali-config/common/includes.chroot/root/
mkdir -p kali-config/common/includes.chroot/usr/share/panter-images/
mkdir -p kali-config/common/package-lists/
mkdir -p kali-config/common/hooks/live/
```

Kontrol
```bash
ls -ld ~/panter-os/kali-config/common/includes.chroot/etc/skel/
ls -ld ~/panter-os/kali-config/common/includes.chroot/root/
ls -ld ~/panter-os/kali-config/common/includes.chroot/usr/share/panter-images/
ls -ld ~/panter-os/kali-config/common/package-lists/
ls -ld ~/panter-os/kali-config/common/hooks/live/
```

### Adım 3: Görsellerin Hazırlanması
Komutları çalıştırmadan önce, kullanacağınız görsellerin ana makinenizin Masaüstü (`~/Desktop/`) dizininde ve aşağıdaki isimlerle hazır olduğundan emin olun:

* `desktop.png` (XFCE Masaüstü arka planı için)
* `login.png` (LightDM giriş ekranı arka planı için)

Görsellerin hazır olduğunu doğruladıktan sonra, proje klasörüne kopyalamak için şu komutları çalıştırın:
```bash
# Masaüstündeki tüm desktop.* ve login.* dosyalarını kopyala
cp ~/Desktop/desktop.* kali-config/common/includes.chroot/usr/share/panter-images/
cp ~/Desktop/login.* kali-config/common/includes.chroot/usr/share/panter-images/
```
Kontrol
```bash
ls -l kali-config/common/includes.chroot/usr/share/panter-images/
file kali-config/common/includes.chroot/usr/share/panter-images/*
```
### Adım 4: Sistem Kimliği (Hostname & MOTD)
Sistem açıldığında terminalde belirecek karşılama mesajını ve makine adını ayarlayın:

```bash
cat << 'EOF' > kali-config/common/includes.chroot/etc/motd
============================================================
* DAĞITIM: PANTER-OS SİBER GÜVENLİK LABORATUVARI OS        *
* DURUM: SİSTEM BAŞARIYLA YÜKLENDİ...                      *
============================================================
EOF
```
```bash
echo "cyber-node-01" > kali-config/common/includes.chroot/etc/hostname
```
```bash
cat << 'EOF' > kali-config/common/includes.chroot/etc/hosts
127.0.0.1   localhost
127.0.1.1   cyber-node-01
::1         localhost ip6-localhost ip6-loopback
ff02::1     ip6-allnodes
ff02::2     ip6-allrouters
EOF
```

Kontrol
```bash
cat kali-config/common/includes.chroot/etc/motd
cat kali-config/common/includes.chroot/etc/hostname
cat kali-config/common/includes.chroot/etc/hosts
```

### Adım 5: Terminal Tasarımı ve Paket Listesi
Özel [SEC-LAB] terminal yapısını ve ISO içine gömülecek laboratuvar araçları ile X11 sanal makine sürücülerini tanımlayın:

```bash
# Normal kullanıcı (kali) için Terminal Tasarımı
cat << 'EOF' > kali-config/common/includes.chroot/etc/skel/.bashrc
export PS1="\[\e[1;31m\][SEC-LAB]\[\e[0m\]-\[\e[1;32m\]\u@\h\[\e[0m\]:\[\e[1;34m\]\w\[\e[0m\]\$ "
EOF
```
```bash
# Root kullanıcısı için Terminal Tasarımı
cat << 'EOF' > kali-config/common/includes.chroot/root/.bashrc
export PS1="\[\e[1;31m\][SEC-LAB]\[\e[0m\]-\[\e[1;32m\]\u@\h\[\e[0m\]:\[\e[1;34m\]\w\[\e[0m\]\$ "
EOF
```
```bash
# Gerekli Araçlar ve VirtualBox Sürücüleri
cat << 'EOF' > kali-config/common/package-lists/panter-tools.list.chroot
nmap
wireshark
tmux
virtualbox-guest-x11
virtualbox-guest-utils
EOF
```

Kontrol
```bash
cat kali-config/common/includes.chroot/etc/skel/.bashrc
cat kali-config/common/includes.chroot/root/.bashrc
cat kali-config/common/package-lists/panter-tools.list.chroot
```

### Adım 6: Otomasyon Betiği (Hook Script)
Derleme işleminin son aşamasında özelleştirmelerin uygulanması için bir hook betiği oluşturun.

```bash
cat << 'EOF' > kali-config/common/hooks/live/99-panter-os-setup.chroot
#!/bin/sh
echo ">>> Applying Panter-OS customizations..."

# ==========================================
# 1. MASAÜSTÜ ARKA PLANI (XFCE)
# ==========================================
# YÖNTEM A: Ezme (Format uyuşmazlığını önlemek için JPG'ye JPG veriyoruz)
rm -f /usr/share/backgrounds/kali/kali-cubes-16x9.jpg
rm -f /usr/share/backgrounds/kali/kali-cubes2-16x9.jpg

cp -f /usr/share/panter-images/desktop.jpg /usr/share/backgrounds/kali/kali-cubes-16x9.jpg
cp -f /usr/share/panter-images/desktop.jpg /usr/share/backgrounds/kali/kali-cubes2-16x9.jpg

# YÖNTEM B: Konfigürasyon (Kural Değiştirme)
update-alternatives --install /usr/share/images/desktop-base/desktop-background desktop-background /usr/share/panter-images/desktop.jpg 999
update-alternatives --set desktop-background /usr/share/panter-images/desktop.jpg

# ==========================================
# 2. GİRİŞ EKRANI (LIGHTDM LOGIN)
# ==========================================
# YÖNTEM A-1: Kök Klasördeki (Backgrounds) Login Dosyalarını Ezme
rm -f /usr/share/backgrounds/kali/login.svg
rm -f /usr/share/backgrounds/kali/login-blurred

cp -f /usr/share/panter-images/login.svg /usr/share/backgrounds/kali/login.svg
cp -f /usr/share/panter-images/login.jpg /usr/share/backgrounds/kali/login-blurred

# YÖNTEM A-2: Tema Klasöründeki Kısayolları Ezme
rm -f /usr/share/desktop-base/kali-theme/login/background
rm -f /usr/share/desktop-base/kali-theme/login/background.svg
rm -f /usr/share/desktop-base/kali-theme/login/background-blurred

# Uzantısız ve blurred dosyaları standart JPG ile eziyoruz
cp -f /usr/share/panter-images/login.jpg /usr/share/desktop-base/kali-theme/login/background
cp -f /usr/share/panter-images/login.jpg /usr/share/desktop-base/kali-theme/login/background-blurred

# SVG dosyasını GERÇEK SVG ile eziyoruz! 
cp -f /usr/share/panter-images/login.svg /usr/share/desktop-base/kali-theme/login/background.svg

# YÖNTEM B: Konfigürasyon (Kural Değiştirme)
mkdir -p /etc/lightdm
cat << 'CONF' > /etc/lightdm/lightdm-gtk-greeter.conf
[greeter]
background=/usr/share/panter-images/login.jpg
CONF

# ==========================================
# 3. KABUK (SHELL) AYARLARI VE İZİNLER
# ==========================================
# ZSH yerine BASH'i Varsayılan Yapma
sed -i 's/DSHELL=\/bin\/zsh/DSHELL=\/bin\/bash/g' /etc/adduser.conf
chsh -s /bin/bash root

chmod -R 755 /usr/share/backgrounds/kali/
chmod -R 755 /usr/share/desktop-base/kali-theme/login/
chmod -R 755 /usr/share/panter-images/

echo ">>> Panter-OS customization completed."
EOF
```
```bash
# Betiğe çalışma izni verin
chmod +x kali-config/common/hooks/live/99-panter-os-setup.chroot
```

Kontrol
```bash
ls -l kali-config/common/hooks/live/99-panter-os-setup.chroot
```

### Adım 7: Derlemeyi Başlatma
Tüm konfigürasyonlar tamamlandıktan sonra XFCE masaüstü varyantını belirterek ISO derleme sürecini başlatın:

```bash
sudo lb clean
sudo ./build.sh --variant xfce && mv images/*.iso images/panter-os.iso
```

Takip/Kontrol
```bash
#derleme esnasında logları takip etmek için farklı bir terminalde aşağıdaki komutu çalıştırın.
tail -f build.log

#iso dosyasının çıktısını buradan kontrol edin.
ls -lh images/panter-os.iso
```


⚠️ Kontrol Edilmesi Gereken Kritik Adımlar (Checklist)

- **RAM:** En az 4 GB
- **Video Memory:** 128 MB
- **Graphics Controller:** VBoxSVGA
- **3D Acceleration:** Disabled
- **Hook Script:** Çalıştırma izni verilmiş olmalı
