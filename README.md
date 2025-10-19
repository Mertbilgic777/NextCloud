# Proje: Oracle Cloud Üzerinde Docker ile Kişisel Nextcloud Kurulumu

Bu doküman, sıfırdan bir bulut sunucu üzerinde, konteyner teknolojisi kullanılarak kişisel bir bulut depolama ve işbirliği platformu olan Nextcloud'un kurulum aşamalarını detaylandırmaktadır.

## 1. Projenin Amacı

Projenin temel amacı, farklı cihazlar (masaüstü, dizüstü bilgisayar, telefon) arasında dosya senkronizasyonu ve uzaktan erişim ihtiyacına, üçüncü parti bulut servislerine bir alternatif olarak, güvenli ve kişisel bir çözüm oluşturmaktır. Veri kontrolünü ve gizliliğini tamamen kullanıcıya bırakarak, açık kaynaklı bir platform olan Nextcloud'un tüm avantajlarından faydalanmak hedeflenmiştir.

## 2. Kullanılan Teknolojiler (Technology Stack)

* **Bulut Sağlayıcı:** Oracle Cloud Infrastructure (OCI) Free Tier
* **İşletim Sistemi:** Ubuntu 22.04 LTS
* **Konteynerizasyon:** Docker & Docker Compose
* **Uygulama:** Nextcloud
* **Veritabanı:** MariaDB
* **Güvenlik Duvarı:** UFW (Uncomplicated Firewall) & OCI Security Lists

## 3. Altyapının Kurulumu ve Yapılandırılması

### 3.1. Oracle Cloud (OCI) Sanal Makine Kurulumu

Projenin temeli, OCI'ın "Her Zaman Ücretsiz" (Always Free) katmanı kullanılarak oluşturulan bir sanal makine (VM) üzerine kurulmuştur. Kurulum sırasında aşağıdaki özelliklere sahip bir instance tercih edilmiştir:
* **Shape:** `VM.Standard.E2.1.Micro`
* **Image:** Ubuntu 22.04
* **Ağ:** Gerekli VCN (Sanal Bulut Ağı) ve Public Subnet oluşturuldu.
* **Erişim:** SSH anahtar tabanlı (key-based) güvenli erişim sağlandı.

### 3.2. Sunucu İlk Hazırlığı ve Güvenlik Ayarları

Sıfır bir Ubuntu sunucusuna ilk SSH bağlantısı yapıldıktan sonra, aşağıdaki temel hazırlık ve güvenlik adımları uygulanmıştır:

1.  **Sistem Güncellemesi:**
    ```bash
    sudo apt update && sudo apt upgrade -y
    ```
2.  **Sudo Yetkili Yeni Kullanıcı Oluşturma:**
    Güvenlik nedeniyle `root` kullanıcısı yerine tüm işlemlerin yapılacağı, `sudo` yetkilerine sahip yeni bir kullanıcı oluşturuldu.
    ```bash
    adduser yeni_kullanici
    usermod -aG sudo yeni_kullanici
    ```
3.  **Temel Güvenlik Duvarı (UFW) Kurulumu:**
    Sadece gerekli portlara izin vermek için UFW etkinleştirildi.
    ```bash
    sudo ufw allow OpenSSH
    sudo ufw allow 80/tcp
    sudo ufw allow 443/tcp
    sudo ufw enable
    ```

### 3.3. Docker ve Docker Compose Kurulumu

Uygulamayı konteynerler içinde çalıştırmak için Docker ve Docker Compose'un en güncel sürümleri, Docker'ın resmi deposu kullanılarak kuruldu.
```bash
# Gerekli bağımlılıklar kuruldu ve Docker'ın GPG anahtarı eklendi.
sudo apt install ca-certificates curl gnupg
# ... (Docker kurulum komutlarının tamamı) ...

# Docker'ı sudo'suz kullanmak için kullanıcı gruba eklendi.
sudo usermod -aG docker ${USER}
