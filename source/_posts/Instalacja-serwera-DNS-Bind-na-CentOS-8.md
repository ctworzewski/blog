---
title: Instalacja serwera DNS Bind na CentOS 8
date: 2021-11-23 06:53:45
tags: dns, administracja, linux, centOS, admin
---

## Przerwa w blogu - kolejna reaktywacja

Postaram się, aby teraz była ciągłość artukułów :)

Ostatnimi czasy bardzo dużo czasu poświęcam na naukę administracji systemiami Linux. Na moim oku są systemy CentOS oraz Ubuntu. Ucze się i konfiguruje różne usługi sieciowe tj. serwery FTP, serwery pocztowe, serwer Apache, stawianie serwerów VPN czy instalacja serwera SQL Server pod Linuxem.
Sprawia mi to olbrzymią przyjemność. Część rzeczy np. mogę wykorzystać już w pracy zwłaszcza z serwerem pocztowym - wiem, gdzie co szukać i czego może być wina.

## Co to jest Bind?

Bind jest serwerem DNS. Jest on jancześciej wybierany przez użytkowników. Jest wkorzystywany w systemach Linux'owych. Serwer Bind zapewnia  poprawne działanie stron www w Internecie. Zamienia adresy IP na przyjazne nazwy i odwrotnie.

## Instalacja Bind

### Podstawowe instalacje programów i usług

* robimy wszelkie potrzebne aktualizacje systemu

```bash
dnf update
```

* instaluje edytor tekstu **nano**

```bash
dnf install nano
```

* przechodzę do instalacji **bind'a**

```bash
dnf install bind bind-utils
```

* uruchamiam moją usługę Bind

```bash
systemctl start named
```

* sprawdzam status usługi - czy działa

```bash
systemctl status named
```

* dodam teraz usługę Bind do autostartu, tak aby uruchamiała się podczas restartu serwera

```bash
systemctl enable named
```

### Konfiguracja serwera Bind

Teraz przechodzimy do podstawowej konfiguracji serwera.

* musimy edytować plik **named.conf**, który znajduje się w katalogu **etc**

```bash
cd /etc/named.conf
```

```bash
nano named.conf
```

* zawartość pliku **named.conf**

musimy zakomentować wskazane linijski

```bash
//      listen-on port 53 { 127.0.0.1; };
//      listen-on-v6 port 53 { ::1; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        secroots-file   "/var/named/data/named.secroots";
        recursing-file  "/var/named/data/named.recursing";
19 //      allow-query     { localhost; };
```

* dopisanie na końcu kawałka kodu do pliku **named.conf**, w którym wskażemy lokalizacje naszego nowego pliku ze strefą DNS. Plik ten nazywa się `it-tworzewski.pl.db`

```bash
zone "it-tworzewski.pl" { type master; file "/var/named/it-tworzewski.pl.db"; };
```

Zapisujemy plik named.conf i wychodzimy

* zanim edytuje plik z moją strefą, musze ten plik utworzyć

```bash
touch /var/named/it-tworzewski.pl.db
```

* edytujemy ten plik strefy

```bash
nano /var/named/it-tworzewski.pl.db
```

* do pliku `nano /var/named/it-tworzewski.pl.db` wklejamy:

```bash
$TTL 3600
@   IN      SOA     dns1.it-tworzewski.pl.      hostmaster.it-tworzewski.pl. (
                                                2021070400
                                                3600
                                                3600
                                                1209600
                                                86400 )
it-tworzewski.pl.         3600    IN      NS      dns1.it-tworzewski.pl.
it-tworzewski.pl.         3600    IN      NS      dns2.it-tworzewski.pl.

it-tworzewski.pl.         3600    IN      A       185.24.219.219
dns1.it-tworzewski.pl.    3600    IN      A       185.24.219.219
dns2.it-tworzewski.pl.    3600    IN      A       185.24.219.219
www                 3600    IN      A       185.24.219.219
```

W wyżej podanym pliku mamy rekordy:

* **NS**

Rekordy serwerów nazw (NS) określają, które serwery przekazują informacje na temat domeny z systemu DNS. Zwykle istnieje podstawowy i dodatkowy rekord serwera nazw dla domeny. Ja użyłem dwóch serwerów  `dns1` oraz `dns2` na tym samym adresie IP. Ze względów bezpieczeństwa warto mieć na innych adresach IP. W przypadku, gdy jeden serwer panie, mamy do dyspozycji drugi (zastępczy)

* **A**

Rekord A (zwany też rekordem adresu lub rekordem hosta) łączy domenę z fizycznym adresem IP serwera

Trzeba pamiętać, aby teraz nazwy serwerów:

``bash
dns1.it-tworzewski.pl
dns2.it-tworzewski.pl
```

podać u rejestratora domeny. Bez podania tych adresów, strona nie będzie widoczna w sieci.

### Jeśli dalej nie działa nasz serwer, to co dalej ?

* zainstaluj firewalla

```bash
dnf install firewalld
```

* uruchom firewall'a

```bash
systemctl start firewalld
systemctl status firewalld
systemctl enable firewalld
```

W tym momencie naprawdopodobniej stracę dostęp do SSH, ponieważ używam innego portu niż standardowy  **port 22**

# Ważne ze względów bezpieczeństwa jest używanie innych portów!

Łączę się do konsoli przez panel klienta firmy hostingowej

* dodaje porty do wyjątku, swój port ssh

ze względów bezpieczeństwa nie napiszę jaki mam :) 

* dodaje domyślny port DNS, czyli port nr 53

```bash
sudo firewall-cmd --permanent --add-port={53/udp,53/tcp}
```

* przeładowuje usługę 

```bash
systemctl restart firewalld
```

## Jak widzimy mamy

![Imgur](https://i.imgur.com/Iytgdbe.png)


# Uwagi mile widziane!!!  :)
