# Bumuo ng sarili mong server ng pagpapadala ng SMTP mail

## pambungad

Maaaring direktang bumili ng SMTP ng mga serbisyo mula sa mga cloud vendor, gaya ng:

* [Amazon SES SMTP](https://docs.aws.amazon.com/ses/latest/dg/send-email-smtp.html)
* [Ali cloud email push](https://www.alibabacloud.com/help/directmail/latest/three-mail-sending-methods)

Maaari ka ring bumuo ng iyong sariling mail server - walang limitasyong pagpapadala, mababang kabuuang gastos.

Sa ibaba, ipinapakita namin ang hakbang-hakbang kung paano bumuo ng aming sariling mail server.

## Pagpili ng server

Ang self-hosted SMTP server ay nangangailangan ng pampublikong IP na may mga port na 25, 456, at 587 na bukas.

Ang mga karaniwang ginagamit na pampublikong ulap ay na-block ang mga port na ito bilang default, at maaaring posible na buksan ang mga ito sa pamamagitan ng pag-isyu ng isang utos sa trabaho, ngunit ito ay napakahirap pagkatapos ng lahat.

Inirerekomenda ko ang pagbili mula sa isang host na nakabukas ang mga port na ito at sumusuporta sa pag-set up ng mga reverse domain name.

Dito, inirerekomenda ko [ang Contabo](https://contabo.com) .

Ang Contabo ay isang hosting provider na nakabase sa Munich, Germany, na itinatag noong 2003 na may napakahusay na presyo.

Kung pipiliin mo ang Euro bilang currency ng pagbili, ang presyo ay magiging mas mura (isang server na may 8GB memory at 4 na CPU ay nagkakahalaga ng humigit-kumulang 529 yuan bawat taon, at ang paunang bayad sa pag-install ay libre para sa isang taon).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/UoAQkwY.webp)

Kapag naglalagay ng order, `prefer AMD` , at ang server na may AMD CPU ay magkakaroon ng mas mahusay na pagganap.

Sa mga sumusunod, kukunin ko ang VPS ng Contabo bilang isang halimbawa upang ipakita kung paano bumuo ng iyong sariling mail server.

## Configuration ng system ng Ubuntu

Ang operating system dito ay Ubuntu 22.04

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/smpIu1F.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/m7Mwjwr.webp)

Kung ang server sa ssh ay nagpapakita `Welcome to TinyCore 13!` (tulad ng ipinapakita sa figure sa ibaba), nangangahulugan ito na hindi pa naka-install ang system. Mangyaring idiskonekta ang ssh at maghintay ng ilang minuto upang mag-log in muli.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/-qKACz9.webp)

Kapag lumabas `Welcome to Ubuntu 22.04.1 LTS` , kumpleto na ang pagsisimula, at maaari kang magpatuloy sa mga sumusunod na hakbang.

### [Opsyonal] Simulan ang development environment

Ang hakbang na ito ay opsyonal.

Para sa kaginhawahan, inilagay ko ang pag-install at pagsasaayos ng system ng ubuntu software sa [github.com/wactax/ops.os/tree/main/ubuntu](https://github.com/wactax/ops.os/tree/main/ubuntu) .

Patakbuhin ang sumusunod na command upang mai-install sa isang pag-click.

```
bash <(curl -s https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

Mga Chinese user, mangyaring gamitin ang sumusunod na command sa halip, at ang wika, time zone, atbp. ay awtomatikong itatakda.

```
CN=1 bash <(curl -s https://ghproxy.com/https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

### Pinapagana ng Contabo ang IPV6

I-enable ang IPV6 para makapagpadala rin ang SMTP ng mga email na may mga IPV6 address.

i-edit ang `/etc/sysctl.conf`

Baguhin o idagdag ang mga sumusunod na linya

```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
```

I-follow up [ang contabo tutorial: Pagdaragdag ng IPv6 connectivity sa iyong server](https://contabo.com/blog/adding-ipv6-connectivity-to-your-server/)

I-edit `/etc/netplan/01-netcfg.yaml` , magdagdag ng ilang linya tulad ng ipinapakita sa figure sa ibaba (Ang Contabo VPS default na configuration file ay mayroon nang mga linyang ito, alisin lamang ang komento sa kanila).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/5MEi41I.webp)

Pagkatapos ay `netplan apply` para magkabisa ang binagong configuration.

Pagkatapos na matagumpay ang pagsasaayos, maaari mong gamitin `curl 6.ipw.cn` upang tingnan ang ipv6 address ng iyong panlabas na network.

## I-clone ang configuration repository ops

```
git clone https://github.com/wactax/ops.soft.git
```

## Bumuo ng isang libreng SSL certificate para sa iyong domain name

Ang pagpapadala ng mail ay nangangailangan ng SSL certificate para sa pag-encrypt at pag-sign.

Ginagamit namin [ang acme.sh](https://github.com/acmesh-official/acme.sh) upang makabuo ng mga sertipiko.

Ang acme.sh ay isang open source na automated certificate signing tool,

Ipasok ang configuration warehouse ops.soft, patakbuhin `./ssl.sh` , at isang `conf` folder ang gagawin sa **itaas na direktoryo** .

Hanapin ang iyong DNS provider mula sa [acme.sh dnsapi](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) , i-edit `conf/conf.sh` .

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Qjq1C1i.webp)

Pagkatapos ay patakbuhin `./ssl.sh 123.com` upang bumuo ng `123.com` at `*.123.com` na mga sertipiko para sa iyong domain name.

Ang unang pagtakbo ay awtomatikong mag-i-install [ng acme.sh](https://github.com/acmesh-official/acme.sh) at magdagdag ng naka-iskedyul na gawain para sa awtomatikong pag-renew. Maaari mong makita `crontab -l` , mayroong isang linya tulad ng sumusunod.

```
52 0 * * * "/mnt/www/.acme.sh"/acme.sh --cron --home "/mnt/www/.acme.sh" > /dev/null
```

Ang path para sa nabuong certificate ay parang `/mnt/www/.acme.sh/123.com_ecc。`

Ang pag-renew ng sertipiko ay tatawag ng `conf/reload/123.com.sh` script, i-edit ang script na ito, maaari kang magdagdag ng mga command tulad ng `nginx -s reload` upang i-refresh ang cache ng certificate ng mga kaugnay na application.

## Bumuo ng SMTP server na may chasquid

[Ang chasquid](https://github.com/albertito/chasquid) ay isang open source SMTP server na nakasulat sa Go language.

Bilang kapalit ng mga sinaunang programa ng mail server tulad ng Postfix at Sendmail, ang chasquid ay mas simple at mas madaling gamitin, at mas madali din ito para sa pangalawang pag-unlad.

Patakbuhin `./chasquid/init.sh 123.com` ay awtomatikong mai-install sa isang pag-click (palitan ang 123.com ng iyong pagpapadala ng domain name).

## I-configure ang Email Signature DKIM

Ginagamit ang DKIM upang magpadala ng mga lagda sa email upang maiwasang ituring ang mga titik bilang spam.

Matapos gumana nang matagumpay ang command, ipo-prompt kang itakda ang DKIM record (tulad ng ipinapakita sa ibaba).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/LJWGsmI.webp)

Magdagdag lang ng TXT record sa iyong DNS (tulad ng ipinapakita sa ibaba).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/0szKWqV.webp)

## Tingnan ang katayuan ng serbisyo at mga log

 `systemctl status chasquid` Tingnan ang katayuan ng serbisyo.

Ang estado ng normal na operasyon ay tulad ng ipinapakita sa figure sa ibaba

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/CAwyY4E.webp)

 Maaaring tingnan `grep chasquid /var/log/syslog` o `journalctl -xeu chasquid` ang log ng error.

## Baliktarin ang configuration ng domain name

Ang reverse domain name ay upang payagan ang IP address na malutas sa kaukulang domain name.

Ang pagtatakda ng reverse domain name ay maaaring maiwasan ang mga email na matukoy bilang spam.

Kapag natanggap ang mail, magsasagawa ang receiving server ng reverse domain name analysis sa IP address ng nagpapadalang server upang kumpirmahin kung ang nagpapadalang server ay may wastong reverse domain name.

Kung ang nagpapadalang server ay walang reverse domain name o kung ang reverse domain name ay hindi tumutugma sa IP address ng nagpapadalang server, maaaring kilalanin ng tatanggap na server ang email bilang spam o tanggihan ito.

Bisitahin ang [https://my.contabo.com/rdns](https://my.contabo.com/rdns) at i-configure tulad ng ipinapakita sa ibaba

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/IIPdBk_.webp)

Pagkatapos itakda ang reverse domain name, tandaan na i-configure ang forward resolution ng domain name na ipv4 at ipv6 sa server.

## I-edit ang hostname ng chasquid.conf

Baguhin `conf/chasquid/chasquid.conf` sa halaga ng reverse domain name.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/6Fw4SQi.webp)

Pagkatapos ay patakbuhin `systemctl restart chasquid` upang i-restart ang serbisyo.

## I-backup ang conf sa git repository

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Fier9uv.webp)

Halimbawa, bina-back ko ang conf folder sa sarili kong proseso ng github gaya ng mga sumusunod

Gumawa muna ng pribadong bodega

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ZSCT1t1.webp)

Ipasok ang conf directory at isumite sa warehouse

```
git init
git add .
git commit -m "init"
git branch -M main
git remote add origin git@github.com:wac-tax-key/conf.git
git push -u origin main
```

## Magdagdag ng nagpadala

tumakbo

```
chasquid-util user-add i@wac.tax
```

Maaaring magdagdag ng nagpadala

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/khHjLof.webp)

### I-verify na ang password ay naitakda nang tama

```
chasquid-util authenticate i@wac.tax --password=xxxxxxx
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/e92JHXq.webp)

Pagkatapos idagdag ang user, maa-update `chasquid/domains/wac.tax/users` , tandaan na isumite ito sa warehouse.

## DNS magdagdag ng SPF record

Ang SPF ( Sender Policy Framework ) ay isang teknolohiya sa pag-verify ng email na ginagamit upang maiwasan ang pandaraya sa email.

Bine-verify nito ang pagkakakilanlan ng isang nagpadala ng mail sa pamamagitan ng pag-check na tumutugma ang IP address ng nagpadala sa mga tala ng DNS ng domain name na sinasabi nito, na pumipigil sa mga manloloko sa pagpapadala ng mga pekeng email.

Ang pagdaragdag ng mga tala ng SPF ay maaaring maiwasan ang mga email na matukoy bilang spam hangga't maaari.

Kung hindi sinusuportahan ng iyong domain name server ang uri ng SPF, magdagdag lang ng tala ng uri ng TXT.

Halimbawa, ang SPF ng `wac.tax` ay ang mga sumusunod

`v=spf1 a mx include:_spf.wac.tax include:_spf.google.com ~all`

SPF para sa `_spf.wac.tax`

`v=spf1 a:smtp.wac.tax ~all`

Tandaan na `include:_spf.google.com` dito, ito ay dahil iko-configure ko `i@wac.tax` bilang address ng pagpapadala sa Google mailbox sa ibang pagkakataon.

## DNS configuration DMARC

Ang DMARC ay ang pagdadaglat ng (Domain-based Message Authentication, Reporting & Conformance).

Ginagamit ito upang makuha ang mga pagtalbog ng SPF (maaaring sanhi ng mga error sa pagsasaayos, o may ibang nagpapanggap na ikaw para magpadala ng spam).

Magdagdag ng TXT record `_dmarc` ,

Ang nilalaman ay ang mga sumusunod

```
v=DMARC1; p=quarantine; fo=1; ruf=mailto:ruf@wac.tax; rua=mailto:rua@wac.tax
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/k44P7O3.webp)

Ang kahulugan ng bawat parameter ay ang mga sumusunod

### p (Patakaran)

Isinasaad kung paano pangasiwaan ang mga email na nabigo sa pag-verify ng SPF (Sender Policy Framework) o DKIM (DomainKeys Identified Mail). Ang p parameter ay maaaring itakda sa isa sa tatlong mga halaga:

* wala: Walang aksyon na ginawa, tanging ang resulta ng pag-verify ang ibabalik sa nagpadala sa pamamagitan ng mekanismo ng pag-uulat ng email.
* Quarantine: Ilagay ang mail na hindi nakapasa sa pag-verify sa folder ng spam, ngunit hindi direktang tatanggihan ang mail.
* tanggihan: Direktang tanggihan ang mga email na nabigo sa pag-verify.

### fo (Mga Pagpipilian sa Pagkabigo)

Tinutukoy ang dami ng impormasyong ibinalik ng mekanismo ng pag-uulat. Maaari itong itakda sa isa sa mga sumusunod na halaga:

* 0: Mag-ulat ng mga resulta ng pagpapatunay para sa lahat ng mga mensahe
* 1: Mag-ulat lamang ng mga mensaheng nabigo sa pag-verify
* d: Iulat lamang ang mga pagkabigo sa pag-verify ng domain name
* s: iulat lamang ang mga pagkabigo sa pag-verify ng SPF
* l: Iulat lamang ang mga pagkabigo sa pag-verify ng DKIM

### rua at ruf

* rua (Pag-uulat ng URI para sa Pinagsama-samang mga ulat): Email address para sa pagtanggap ng pinagsama-samang mga ulat
* ruf (Pag-uulat ng URI para sa Forensic na mga ulat): email address upang makatanggap ng mga detalyadong ulat

## Magdagdag ng mga tala ng MX upang ipasa ang mga email sa Google Mail

Dahil hindi ako makahanap ng libreng corporate mailbox na sumusuporta sa mga unibersal na address (Catch-All, maaaring makatanggap ng anumang mga email na ipinadala sa domain name na ito, nang walang mga paghihigpit sa mga prefix), gumamit ako ng chasquid para ipasa ang lahat ng email sa aking Gmail mailbox.

**Kung mayroon kang sariling bayad na mailbox ng negosyo, mangyaring huwag baguhin ang MX at laktawan ang hakbang na ito.**

I-edit `conf/chasquid/domains/wac.tax/aliases` , itakda ang pagpapasa ng mailbox

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/OBDl2gw.webp)

`*` ay nagpapahiwatig ng lahat ng mga email, `i` ay ang email address na prefix ng nagpapadalang user na ginawa sa itaas. Upang ipasa ang mail, kailangang magdagdag ng linya ang bawat user.

Pagkatapos ay idagdag ang MX record (Direkta akong tumuturo sa address ng reverse domain name dito, tulad ng ipinapakita sa unang linya sa figure sa ibaba).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/7__KrU8.webp)

Pagkatapos makumpleto ang configuration, maaari kang gumamit ng iba pang mga email address upang magpadala ng mga email sa `i@wac.tax` at `any123@wac.tax` upang makita kung makakatanggap ka ng mga email sa Gmail.

Kung hindi, suriin ang chasquid log ( `grep chasquid /var/log/syslog` ).

## Magpadala ng email sa i@wac.tax gamit ang Google Mail

Pagkatapos matanggap ng Google Mail ang mail, natural na umaasa akong tumugon gamit `i@wac.tax` sa halip na i.wac.tax@gmail.com.

Bisitahin ang [https://mail.google.com/mail/u/1/#settings/accounts](https://mail.google.com/mail/u/1/#settings/accounts) at i-click ang "Magdagdag ng isa pang email address."

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/PAvyE3C.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/_OgLsPT.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/XIUf6Dc.webp)

Pagkatapos, ilagay ang verification code na natanggap ng email kung saan ipinasa.

Sa wakas, maaari itong itakda bilang default na address ng nagpadala (kasama ang opsyong tumugon sa parehong address).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/a95dO60.webp)

Sa ganitong paraan, natapos na namin ang pagtatatag ng SMTP mail server at sa parehong oras ay gumagamit ng Google Mail upang magpadala at tumanggap ng mga email.

## Magpadala ng pansubok na email para tingnan kung matagumpay ang configuration

Ipasok `ops/chasquid`

`direnv allow` na mag-install ng mga dependencies (na-install ang direnv sa nakaraang proseso ng pagsisimula ng isang key at naidagdag ang isang hook sa shell)

tapos tumakbo

```
user=i@wac.tax pass=xxxx to=iuser.link@gmail.com ./sendmail.coffee
```

Ang kahulugan ng mga parameter ay ang mga sumusunod

* user: SMTP username
* pass: SMTP password
* sa: tatanggap

Maaari kang magpadala ng pansubok na email.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ae1iWyM.webp)

Inirerekomenda na gamitin ang Gmail upang makatanggap ng mga pansubok na email upang suriin kung matagumpay ang mga pagsasaayos.

### TLS standard encryption

Gaya ng ipinapakita sa figure sa ibaba, mayroong maliit na lock na ito, na nangangahulugang matagumpay na pinagana ang SSL certificate.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/SrdbAwh.webp)

Pagkatapos ay i-click ang "Ipakita ang Orihinal na Email"

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/qQQsdxg.webp)

### DKIM

Gaya ng ipinapakita sa figure sa ibaba, ipinapakita ng orihinal na pahina ng mail ng Gmail ang DKIM, na nangangahulugang matagumpay ang configuration ng DKIM.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/an6aXK6.webp)

Suriin ang Natanggap sa header ng orihinal na email, at makikita mo na ang address ng nagpadala ay IPV6, na nangangahulugang matagumpay na na-configure ang IPV6.
