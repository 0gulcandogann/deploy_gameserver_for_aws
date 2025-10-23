# ☁️ Deploying Minecraft Server to AWS GameLift via Ansible Automation Platform

Bu repo, **Ansible Automation Platform (AAP / AWX)** üzerinden **Vanilla Minecraft sunucusunu AWS GameLift’e deploy etmek** için hazırlanmış bir playbook içeriyor.  
Amaç, tüm AWS tarafındaki işlemleri (IAM, S3, GameLift Fleet, Autoscaling) tamamen otomatik hale getirmek.  
Yani `ansible-playbook` komutlarıyla uğraşmadan, işleri AAP’in üstlenmesini sağlıyoruz.

---

## 🎯 Ne Yaptık

Bu playbook temelde şunları yapıyor:

- AWS üzerinde GameLift için gerekli **IAM rollerini ve politikaları** oluşturur.  
- **S3 bucket’ı** açar ve Minecraft server build’ini oraya yükler.  
- GameLift’e **build** kaydını ekler, **Fleet** oluşturur.  
- Son olarak, **Autoscaling** politikalarını ayarlar.  

Yani “lokalde 20 komutla yapacağın işi” AAP’te tek bir Job Template haline getiriyoruz.

---

## ⚙️ Gereksinimler

Bu setup’ı çalıştırmak için şunlara ihtiyacın var:

- AWS hesabı (GameLift aktif olmalı)
- IAM kullanıcı/rol (gerekli izinler: `gamelift:*`, `iam:*`, `ec2:*`, `s3:*`, `autoscaling:*`)
- AAP (Ansible Automation Platform) ya da açık kaynak AWX kurulumu
- AWS CLI ile test edilmiş bir credential (AAP içine ekleyeceğiz)

---

## 🧠 Kurulum Adımları

### 1️⃣ Reponun İçeriğini İçeri Aktar

```bash
git clone https://github.com/<your-repo>/minecraft-gamelift.git
```

Ardından, `main.yaml` ve `core/` dizinlerini olduğu gibi AAP’e upload et.  
AWX / AAP’te bir **Project** oluştur:
- Type: Git
- SCM URL: repo linki
- Branch: main (veya senin branch’in)

---

### 2️⃣ AWS Credential Ekle

AAP Dashboard’da:  
**Credentials → Add → Amazon Web Services**  
- Access Key ID
- Secret Access Key
- Default region (örnek: `eu-central-1`)

Sonra bu credential’ı playbook’la ilişkilendireceğiz.

---

### 3️⃣ Job Template Oluştur

AAP’te yeni bir Job Template ekle:

| Alan | Değer |
|------|--------|
| **Name** | Minecraft AWS GameLift Deploy |
| **Inventory** | localhost |
| **Project** | minecraft-gamelift |
| **Playbook** | main.yaml |
| **Credentials** | AWS Credentials (az önce eklediğin) |

İstersen “Prompt on launch” seçeneğini aktif bırak, böylece her run’da parametreleri değiştirebilirsin.

---

### 4️⃣ Playbook’u Çalıştır

Run tuşuna bastığında:
- AAP, senin playbook’unu alır.  
- AWS tarafında IAM, S3, Build, Fleet ve Autoscaling’i sırasıyla oluşturur.  
- Tüm log’lar AAP Job Output kısmında görünür.

Fleet’in **“Active”** hale gelmesi birkaç dakika sürebilir.  
Sonrasında AWS GameLift console’a gidip build’in başarıyla deploy edildiğini görebilirsin.

---

## 🔧 Temizlik (Cleanup)

AWS maliyetlerinden kaçınmak için:
- Eğer `destroy.yaml` varsa AAP üzerinden aynı şekilde çalıştır.  
- Yoksa AWS Console’dan oluşturulan kaynakları (Fleet, Build, IAM, S3) manuel temizle.

---

## 💡 Notlar

- Şu anda **Vanilla Minecraft server** için ayarlandı.  
- `core.roles.build` kısmını değiştirerek **Paper**, **Spigot** veya **Fabric** build’leri ekleyebilirsin.  
- AAP’te bir **Schedule** ekleyip otomatik olarak deploy veya cleanup yaptırmak da mümkün.  
- Eğer daha ileri seviye entegrasyon istiyorsan, bu yapı kolayca **Ansible Operator**’a dönüştürülebilir.

---

## 📚 Referanslar

- [AWS GameLift Documentation](https://docs.aws.amazon.com/gamelift/latest/developerguide/)
- [Ansible Automation Platform Docs](https://docs.ansible.com/automation-controller/latest/html/userguide/)
- [Minecraft Dedicated Server Setup](https://minecraft.fandom.com/wiki/Tutorials/Setting_up_a_server)
