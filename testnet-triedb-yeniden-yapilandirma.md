# Monad Testnet — TrieDB Yeniden Yapılandırma

## 1. Servisleri Durdur

```bash
systemctl stop monad-bft monad-execution monad-rpc monad-mpt otelcol
```

---

## 2. TrieDB Sürücüsünü Tespit Et

Mevcut symlink'e bakarak hangi fiziksel sürücünün kullanıldığını teyit et:

```bash
ls -la /dev/triedb
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT,PARTUUID | grep -v loop
```

Çıktıda `/dev/triedb` hangi cihaza işaret ediyorsa (örn. `nvme0n1p1`, `nvme1n1p1`) bir üst cihaz adını (`nvme0n1`, `nvme1n1` vb.) not al ve aşağıdaki adımlarda `TRIEDB_DEV` değişkeni olarak kullan:

```bash
# Cihaz adını buraya gir (örn. nvme0n1, nvme1n1, nvme2n1 ...)
TRIEDB_DEV=
```

---

## 3. Diski Formatla ve Bölümlendir

```bash
nvme format --lbaf=0 --force /dev/$TRIEDB_DEV
parted /dev/$TRIEDB_DEV mklabel gpt
parted /dev/$TRIEDB_DEV mkpart triedb 0% 100%
```

---

## 4. udev Kurallarını Yapılandır

```bash
PARTUUID=$(lsblk -o PARTUUID /dev/${TRIEDB_DEV}p1 | grep -v PARTUUID)
echo "ENV{ID_PART_ENTRY_UUID}==\"$PARTUUID\", MODE=\"0666\", SYMLINK+=\"triedb\"" \
  > /etc/udev/rules.d/99-triedb.rules

udevadm control --reload
udevadm trigger
```

---

## 5. Symlink'i Manuel Oluştur

```bash
rm -f /dev/triedb
ln -sf /dev/${TRIEDB_DEV}p1 /dev/triedb
chmod 666 /dev/triedb
```

---

## 6. Hugepages'i Başlat

```bash
systemctl start set-hugepages
```

---

## 7. MPT'yi Başlat (monad kullanıcısı olarak)

```bash
su - monad
source .env
monad-mpt --storage /dev/triedb --truncate --yes
exit
```

---

## 8. En Son Forkpoint'i İndir

```bash
su - monad
source .env
MF_BUCKET=https://bucket.monadinfra.com
curl -sSL $MF_BUCKET/scripts/testnet/download-forkpoint.sh | bash
exit
```

---

## 9. Validator Konfigürasyonunu Güncelle

```bash
su - monad
curl -o monad-bft/config/validators.toml \
  https://bucket.monadinfra.com/validators/testnet/validators.toml
  exit
```

---

## 10. Servisleri Sırayla Başlat

```bash
systemctl start monad-mpt
sleep 25
systemctl start monad-bft
sleep 25
systemctl start monad-execution monad-rpc
systemctl start otelcol
```
