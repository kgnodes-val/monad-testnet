# Monad Testnet Validator — TrieDB Yeniden Yapılandırma Rehberi

Bu rehber, aktif çalışan bir Monad testnet validator node'unda TrieDB sürücüsünü sıfırdan yeniden yapılandırmak için hazırlanmıştır.

## Ne Zaman Kullanılır?

- TrieDB sürücüsünde bozulma veya veri tutarsızlığı tespit edildiğinde
- Node'un yeni bir forkpoint'ten yeniden sync edilmesi gerektiğinde
- Monad ekibinin TrieDB yeniden yapılandırması talep ettiği durumlarda

## Ön Koşullar

- Sunucuya root erişimi
- `monad` kullanıcısı ve home dizininde `.env` dosyası mevcut olmalı
- TrieDB için ayrılmış bir NVMe sürücü (`/dev/triedb` symlink'i tanımlı olmalı)
- İnternet bağlantısı (forkpoint ve validators.toml indirmek için)

## Önemli Uyarılar

> **Yanlış sürücüyü formatlamak işletim sistemini silebilir.**
> Adım 2'deki sürücü tespitini mutlaka yapın ve `TRIEDB_DEV` değişkenini
> doğruladıktan sonra devam edin.

- Tüm komutlar sırasıyla çalıştırılmalıdır, adım atlanmamalıdır
- `systemctl` komutları root yetkisi gerektirir (`sudo -i` ile geçin)
- `monad-mpt` ve forkpoint adımları `monad` kullanıcısı olarak çalıştırılmalıdır
- Servisler durdurulmadan disk işlemlerine geçilmemelidir

## Tahmini Süre

Forkpoint indirme hızına bağlı olarak **30 dakika — birkaç saat** arasında değişebilir.

## Kaynak

Monad resmi dökümanı: [docs.monad.xyz](https://docs.monad.xyz)
