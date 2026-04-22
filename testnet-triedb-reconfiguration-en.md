# Monad Testnet — TrieDB Reconfiguration

## 1. Stop All Services

```bash
systemctl stop monad-bft monad-execution monad-rpc monad-mpt otelcol
```

---

## 2. Identify the TrieDB Drive

Check the existing symlink to confirm which physical drive is being used:

```bash
ls -la /dev/triedb
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT,PARTUUID | grep -v loop
```

Note the parent device name that `/dev/triedb` points to (e.g. `nvme0n1p1` → parent is `nvme0n1`) and set it as the `TRIEDB_DEV` variable for all subsequent steps:

```bash
# Enter your device name here (e.g. nvme0n1, nvme1n1, nvme2n1 ...)
TRIEDB_DEV=
```

---

## 3. Format and Partition the Drive

```bash
nvme format --lbaf=0 --force /dev/$TRIEDB_DEV
parted /dev/$TRIEDB_DEV mklabel gpt
parted /dev/$TRIEDB_DEV mkpart triedb 0% 100%
```

---

## 4. Configure udev Rules

```bash
PARTUUID=$(lsblk -o PARTUUID /dev/${TRIEDB_DEV}p1 | grep -v PARTUUID)
echo "ENV{ID_PART_ENTRY_UUID}==\"$PARTUUID\", MODE=\"0666\", SYMLINK+=\"triedb\"" \
  > /etc/udev/rules.d/99-triedb.rules

udevadm control --reload
udevadm trigger
```

---

## 5. Create Symlink Manually

```bash
rm -f /dev/triedb
ln -sf /dev/${TRIEDB_DEV}p1 /dev/triedb
chmod 666 /dev/triedb
```

---

## 6. Start Hugepages

```bash
systemctl start set-hugepages
```

---

## 7. Initialize MPT (as monad user)

```bash
su - monad
source .env
monad-mpt --storage /dev/triedb --truncate --yes
exit
```

---

## 8. Download the Latest Forkpoint

```bash
su - monad
source .env
MF_BUCKET=https://bucket.monadinfra.com
curl -sSL $MF_BUCKET/scripts/testnet-2/download-forkpoint.sh | bash
exit
```

---

## 9. Update Validator Configuration

```bash
curl -o /home/monad/monad-bft/config/validators.toml \
  https://bucket.monadinfra.com/validators/testnet-2/validators.toml
```

---

## 10. Start Services in Order

```bash
systemctl start monad-mpt
sleep 25
systemctl start monad-bft
sleep 25
systemctl start monad-execution monad-rpc
systemctl start otelcol
```
