---
layout: post
title: "Intermediate Certificates with Sendmail"
date: 2014-04-11 14:00:05 -0700
comments: true
categories:
author: Ken Berland
---
We got a certificate for our mailserver from Comodo. We constructed a .pem file in the usual [way][1] and set sendmail to eat the file in the `sendmail.cf`. The operative lines are these:

    # CA directory
    O CACertPath=/etc/mail/certs
    # CA file
    O CACertFile=/etc/mail/ssl/ca-bundle.crt
    # Server Cert
    O ServerCertFile=/etc/mail/ssl/sendmail.pem
    # Server private key
    O ServerKeyFile=/etc/mail/ssl/sendmail.pem
    # Client Cert
    O ClientCertFile=/etc/mail/ssl/sendmail.pem
    # Client private key
    O ClientKeyFile=/etc/mail/ssl/sendmail.pem
    

However, after testing the connection, nobody was happy:

    $ openssl s_client -starttls smtp -crlf -connect mail.hero.com:587
    CONNECTED(00000004)
    depth=0 OU = Domain Control Validated, OU = PositiveSSL, CN = mail.hero.com
    verify error:num=20:unable to get local issuer certificate
    verify return:1
    depth=0 OU = Domain Control Validated, OU = PositiveSSL, CN = mail.hero.com
    verify error:num=27:certificate not trusted
    verify return:1
    depth=0 OU = Domain Control Validated, OU = PositiveSSL, CN = mail.hero.com
    verify error:num=21:unable to verify the first certificate
    verify return:1
    ---
    Certificate chain
     0 s:/OU=Domain Control Validated/OU=PositiveSSL/CN=mail.hero.com
       i:/C=GB/ST=Greater Manchester/L=Salford/O=COMODO CA Limited/CN=PositiveSSL CA 2
    ---
    Server certificate
    -----BEGIN CERTIFICATE-----
    MIIE/DCCA+SgAwIBAgIRANbFkP3QD8z1aKuBcP950t8wDQYJKoZIhvcNAQEFBQAw
    czELMAkGA1UEBhMCR0IxGzAZBgNVBAgTEkdyZWF0ZXIgTWFuY2hlc3RlcjEQMA4G
    A1UEBxMHU2FsZm9yZDEaMBgGA1UEChMRQ09NT0RPIENBIExpbWl0ZWQxGTAXBgNV
    BAMTEFBvc2l0aXZlU1NMIENBIDIwHhcNMTQwNDA2MDAwMDAwWhcNMTUwNDA2MjM1
    OTU5WjBRMSEwHwYDVQQLExhEb21haW4gQ29udHJvbCBWYWxpZGF0ZWQxFDASBgNV
    BAsTC1Bvc2l0aXZlU1NMMRYwFAYDVQQDEw1tYWlsLmhlcm8uY29tMIIBIjANBgkq
    hkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAwpbNmIuaptpTjg3GkAoSt4i1DYrIMVN5
    VP5/TkKZz4lDLUbtZOm/+LE0dD00jSYHFiY9EC1u8V/3jr4hLHLKa6PKDwWaqyPN
    MGpHsJqfeXdiJOR8WrdB7VdHE86TYWk4G54kk39JmF9OJ3cRwvLm3gz6dDMO4JsS
    lNhrkrsakHyHKlxuNo+6R6MPiDSPFYzV5TlTzc3wBsZeCnTFbRby3IikhQ7ZySWD
    nslD+SOI9vnyxTV5xbKCV234SzJQRod9kgeZf9TSoXLUZoUzrkpea/uhcMvdLQXB
    fVm2VthjgRXKDzpFAoyHPZl40IwDuKFEdwBQ6i/iopMYc1cojvCCiQIDAQABo4IB
    qzCCAacwHwYDVR0jBBgwFoAUmeRAX2sUXj4F2d3TY1T8Yrj3AKwwHQYDVR0OBBYE
    FNocZVZ3U0rpEid5rfYyZVldMTLbMA4GA1UdDwEB/wQEAwIFoDAMBgNVHRMBAf8E
    AjAAMB0GA1UdJQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDAjBQBgNVHSAESTBHMDsG
    CysGAQQBsjEBAgIHMCwwKgYIKwYBBQUHAgEWHmh0dHA6Ly93d3cucG9zaXRpdmVz
    c2wuY29tL0NQUzAIBgZngQwBAgEwOwYDVR0fBDQwMjAwoC6gLIYqaHR0cDovL2Ny
    bC5jb21vZG9jYS5jb20vUG9zaXRpdmVTU0xDQTIuY3JsMGwGCCsGAQUFBwEBBGAw
    XjA2BggrBgEFBQcwAoYqaHR0cDovL2NydC5jb21vZG9jYS5jb20vUG9zaXRpdmVT
    U0xDQTIuY3J0MCQGCCsGAQUFBzABhhhodHRwOi8vb2NzcC5jb21vZG9jYS5jb20w
    KwYDVR0RBCQwIoINbWFpbC5oZXJvLmNvbYIRd3d3Lm1haWwuaGVyby5jb20wDQYJ
    KoZIhvcNAQEFBQADggEBAMLYKhr7EFl0iXioAbvfyqRBhadbCEb0jRNkzokCFDM6
    Umlfi+z/ACvx4iHZWxz1z3+XReGHjfQMDEQzK+JKqTLu1A3ZHJCTfGO031nEnXi7
    cEjIrTo1kGolC+TWT5CihniWl7IQNRGlbFqtfaAKKjzFppwwwiy3gewlMJfa4m9x
    9ruqp1neI8DGswdmYKyFMcOEyxcIneDacd98Qoldq60zisuQOC8MsOylCee2n/Yg
    7eGBD59mHlB1RfrdccNHMAMJiJ4F0b2KMJTtENFCFNtMnCeDPussv8mcgxD0OEPi
    2XraR+7ORQrdSlu63sSDqrg4bukSNh2Ve4DLnt1rBfs=
    -----END CERTIFICATE-----
    subject=/OU=Domain Control Validated/OU=PositiveSSL/CN=mail.hero.com
    issuer=/C=GB/ST=Greater Manchester/L=Salford/O=COMODO CA Limited/CN=PositiveSSL CA 2
    ---
    Acceptable client certificate CA names
    /C=US/O=RSA Data Security, Inc./OU=Secure Server Certification Authority
    /C=US/O=GTE Corporation/CN=GTE CyberTrust Root
    /C=US/O=GTE Corporation/OU=GTE CyberTrust Solutions, Inc./CN=GTE CyberTrust Global Root
    /C=ZA/ST=Western Cape/L=Cape Town/O=Thawte Consulting/OU=Certification Services Division/CN=Thawte Personal Basic CA/emailAddress=personal-basic@thawte.com
    /C=ZA/ST=Western Cape/L=Cape Town/O=Thawte Consulting/OU=Certification Services Division/CN=Thawte Personal Premium CA/emailAddress=personal-premium@thawte.com
    /C=ZA/ST=Western Cape/L=Cape Town/O=Thawte Consulting/OU=Certification Services Division/CN=Thawte Personal Freemail CA/emailAddress=personal-freemail@thawte.com
    /C=ZA/ST=Western Cape/L=Cape Town/O=Thawte Consulting cc/OU=Certification Services Division/CN=Thawte Server CA/emailAddress=server-certs@thawte.com
    /C=ZA/ST=Western Cape/L=Cape Town/O=Thawte Consulting cc/OU=Certification Services Division/CN=Thawte Premium Server CA/emailAddress=premium-server@thawte.com
    /C=US/O=Equifax/OU=Equifax Secure Certificate Authority
    /C=US/ST=DC/L=Washington/O=ABA.ECOM, INC./CN=ABA.ECOM Root CA/emailAddress=admin@digsigtrust.com
    /C=US/O=Digital Signature Trust Co./OU=DSTCA E1
    /C=US/O=Digital Signature Trust Co./OU=DSTCA E2
    /C=us/ST=Utah/L=Salt Lake City/O=Digital Signature Trust Co./OU=DSTCA X1/CN=DST RootCA X1/emailAddress=ca@digsigtrust.com
    /C=us/ST=Utah/L=Salt Lake City/O=Digital Signature Trust Co./OU=DSTCA X2/CN=DST RootCA X2/emailAddress=ca@digsigtrust.com
    /C=US/O=VeriSign, Inc./OU=Class 1 Public Primary Certification Authority
    /C=US/O=VeriSign, Inc./OU=Class 2 Public Primary Certification Authority
    /C=US/O=VeriSign, Inc./OU=Class 3 Public Primary Certification Authority
    /C=US/O=VeriSign, Inc./OU=Class 1 Public Primary Certification Authority - G2/OU=(c) 1998 VeriSign, Inc. - For authorized use only/OU=VeriSign Trust Network
    /C=US/O=VeriSign, Inc./OU=Class 2 Public Primary Certification Authority - G2/OU=(c) 1998 VeriSign, Inc. - For authorized use only/OU=VeriSign Trust Network
    /C=US/O=VeriSign, Inc./OU=Class 3 Public Primary Certification Authority - G2/OU=(c) 1998 VeriSign, Inc. - For authorized use only/OU=VeriSign Trust Network
    /C=US/O=VeriSign, Inc./OU=Class 4 Public Primary Certification Authority - G2/OU=(c) 1998 VeriSign, Inc. - For authorized use only/OU=VeriSign Trust Network
    /C=BE/O=GlobalSign nv-sa/OU=Root CA/CN=GlobalSign Root CA
    /OU=GlobalSign Root CA - R2/O=GlobalSign/CN=GlobalSign
    /L=ValiCert Validation Network/O=ValiCert, Inc./OU=ValiCert Class 1 Policy Validation Authority/CN=http://www.valicert.com//emailAddress=info@valicert.com
    /L=ValiCert Validation Network/O=ValiCert, Inc./OU=ValiCert Class 2 Policy Validation Authority/CN=http://www.valicert.com//emailAddress=info@valicert.com
    /L=ValiCert Validation Network/O=ValiCert, Inc./OU=ValiCert Class 3 Policy Validation Authority/CN=http://www.valicert.com//emailAddress=info@valicert.com
    /C=US/O=VeriSign, Inc./OU=VeriSign Trust Network/OU=(c) 1999 VeriSign, Inc. - For authorized use only/CN=VeriSign Class 1 Public Primary Certification Authority - G3
    /C=US/O=VeriSign, Inc./OU=VeriSign Trust Network/OU=(c) 1999 VeriSign, Inc. - For authorized use only/CN=VeriSign Class 2 Public Primary Certification Authority - G3
    /C=US/O=VeriSign, Inc./OU=VeriSign Trust Network/OU=(c) 1999 VeriSign, Inc. - For authorized use only/CN=VeriSign Class 3 Public Primary Certification Authority - G3
    /C=US/O=VeriSign, Inc./OU=VeriSign Trust Network/OU=(c) 1999 VeriSign, Inc. - For authorized use only/CN=VeriSign Class 4 Public Primary Certification Authority - G3
    /C=US/O=Entrust.net/OU=www.entrust.net/CPS incorp. by ref. (limits liab.)/OU=(c) 1999 Entrust.net Limited/CN=Entrust.net Secure Server Certification Authority
    /C=US/O=Entrust.net/OU=www.entrust.net/Client_CA_Info/CPS incorp. by ref. limits liab./OU=(c) 1999 Entrust.net Limited/CN=Entrust.net Client Certification Authority
    /O=Entrust.net/OU=www.entrust.net/CPS_2048 incorp. by ref. (limits liab.)/OU=(c) 1999 Entrust.net Limited/CN=Entrust.net Certification Authority (2048)
    /C=IE/O=Baltimore/OU=CyberTrust/CN=Baltimore CyberTrust Root
    /C=US/O=Equifax Secure Inc./CN=Equifax Secure Global eBusiness CA-1
    /C=US/O=Equifax Secure Inc./CN=Equifax Secure eBusiness CA-1
    /C=US/O=Equifax Secure/OU=Equifax Secure eBusiness CA-2
    /C=US/O=VISA/OU=Visa International Service Association/CN=GP Root 2
    /C=WW/O=beTRUSTed/CN=beTRUSTed Root CAs/CN=beTRUSTed Root CA
    /C=SE/O=AddTrust AB/OU=AddTrust TTP Network/CN=AddTrust Class 1 CA Root
    /C=SE/O=AddTrust AB/OU=AddTrust External TTP Network/CN=AddTrust External CA Root
    /C=SE/O=AddTrust AB/OU=AddTrust TTP Network/CN=AddTrust Public CA Root
    /C=SE/O=AddTrust AB/OU=AddTrust TTP Network/CN=AddTrust Qualified CA Root
    /O=VeriSign, Inc./OU=VeriSign Trust Network/OU=Terms of use at https://www.verisign.com/rpa (c)00/CN=VeriSign Time Stamping Authority CA
    /C=ZA/ST=Western Cape/L=Durbanville/O=Thawte/OU=Thawte Certification/CN=Thawte Timestamping CA
    /O=Entrust.net/OU=www.entrust.net/SSL_CPS incorp. by ref. (limits liab.)/OU=(c) 2000 Entrust.net Limited/CN=Entrust.net Secure Server Certification Authority
    /O=Entrust.net/OU=www.entrust.net/GCCA_CPS incorp. by ref. (limits liab.)/OU=(c) 2000 Entrust.net Limited/CN=Entrust.net Client Certification Authority
    /C=US/O=Entrust, Inc./OU=www.entrust.net/CPS is incorporated by reference/OU=(c) 2006 Entrust, Inc./CN=Entrust Root Certification Authority
    /C=US/O=AOL Time Warner Inc./OU=America Online Inc./CN=AOL Time Warner Root Certification Authority 1
    /C=US/O=AOL Time Warner Inc./OU=America Online Inc./CN=AOL Time Warner Root Certification Authority 2
    /O=beTRUSTed/OU=beTRUSTed Root CAs/CN=beTRUSTed Root CA-Baltimore Implementation
    /O=beTRUSTed/OU=beTRUSTed Root CAs/CN=beTRUSTed Root CA - Entrust Implementation
    /O=beTRUSTed/OU=beTRUSTed Root CAs/CN=beTRUSTed Root CA - RSA Implementation
    /O=RSA Security Inc/OU=RSA Security 2048 V3
    /O=RSA Security Inc/OU=RSA Security 1024 V3
    /C=US/O=GeoTrust Inc./CN=GeoTrust Global CA
    /C=US/O=GeoTrust Inc./CN=GeoTrust Global CA 2
    /C=US/O=GeoTrust Inc./CN=GeoTrust Universal CA
    /C=US/O=GeoTrust Inc./CN=GeoTrust Universal CA 2
    /C=US/ST=UT/L=Salt Lake City/O=The USERTRUST Network/OU=http://www.usertrust.com/CN=UTN-USERFirst-Network Applications
    /C=US/O=America Online Inc./CN=America Online Root Certification Authority 1
    /C=US/O=America Online Inc./CN=America Online Root Certification Authority 2
    /C=US/O=VISA/OU=Visa International Service Association/CN=Visa eCommerce Root
    /C=DE/ST=Hamburg/L=Hamburg/O=TC TrustCenter for Security in Data Networks GmbH/OU=TC TrustCenter Class 2 CA/emailAddress=certificate@trustcenter.de
    /C=DE/ST=Hamburg/L=Hamburg/O=TC TrustCenter for Security in Data Networks GmbH/OU=TC TrustCenter Class 3 CA/emailAddress=certificate@trustcenter.de
    /C=PL/O=Unizeto Sp. z o.o./CN=Certum CA
    /C=GB/ST=Greater Manchester/L=Salford/O=Comodo CA Limited/CN=AAA Certificate Services
    /C=GB/ST=Greater Manchester/L=Salford/O=Comodo CA Limited/CN=Secure Certificate Services
    /C=GB/ST=Greater Manchester/L=Salford/O=Comodo CA Limited/CN=Trusted Certificate Services
    /C=ES/ST=Barcelona/L=Barcelona/O=IPS Internet publishing Services s.l./O=ips@mail.ips.es C.I.F. B-60929452/OU=IPS CA Chained CAs Certification Authority/CN=IPS CA Chained CAs Certification Authority/emailAddress=ips@mail.ips.es
    /C=ES/ST=Barcelona/L=Barcelona/O=IPS Internet publishing Services s.l./O=ips@mail.ips.es C.I.F. B-60929452/OU=IPS CA CLASE1 Certification Authority/CN=IPS CA CLASE1 Certification Authority/emailAddress=ips@mail.ips.es
    /C=ES/ST=Barcelona/L=Barcelona/O=IPS Internet publishing Services s.l./O=ips@mail.ips.es C.I.F. B-60929452/OU=IPS CA CLASE3 Certification Authority/CN=IPS CA CLASE3 Certification Authority/emailAddress=ips@mail.ips.es
    /C=ES/ST=Barcelona/L=Barcelona/O=IPS Internet publishing Services s.l./O=ips@mail.ips.es C.I.F. B-60929452/OU=IPS CA CLASEA1 Certification Authority/CN=IPS CA CLASEA1 Certification Authority/emailAddress=ips@mail.ips.es
    /C=ES/ST=Barcelona/L=Barcelona/O=IPS Internet publishing Services s.l./O=ips@mail.ips.es C.I.F. B-60929452/OU=IPS CA CLASEA3 Certification Authority/CN=IPS CA CLASEA3 Certification Authority/emailAddress=ips@mail.ips.es
    /C=ES/ST=BARCELONA/L=BARCELONA/O=IPS Seguridad CA/OU=Certificaciones/CN=IPS SERVIDORES/emailAddress=ips@mail.ips.es
    /C=ES/ST=Barcelona/L=Barcelona/O=IPS Internet publishing Services s.l./O=ips@mail.ips.es C.I.F. B-60929452/OU=IPS CA Timestamping Certification Authority/CN=IPS CA Timestamping Certification Authority/emailAddress=ips@mail.ips.es
    /C=BM/O=QuoVadis Limited/OU=Root Certification Authority/CN=QuoVadis Root Certification Authority
    /C=BM/O=QuoVadis Limited/CN=QuoVadis Root CA 2
    /C=BM/O=QuoVadis Limited/CN=QuoVadis Root CA 3
    /C=JP/O=SECOM Trust.net/OU=Security Communication RootCA1
    /C=FI/O=Sonera/CN=Sonera Class1 CA
    /C=FI/O=Sonera/CN=Sonera Class2 CA
    /C=NL/O=Staat der Nederlanden/CN=Staat der Nederlanden Root CA
    /C=DK/O=TDC Internet/OU=TDC Internet Root CA
    /C=DK/O=TDC/CN=TDC OCES CA
    /C=US/ST=UT/L=Salt Lake City/O=The USERTRUST Network/OU=http://www.usertrust.com/CN=UTN - DATACorp SGC
    /C=US/ST=UT/L=Salt Lake City/O=The USERTRUST Network/OU=http://www.usertrust.com/CN=UTN-USERFirst-Client Authentication and Email
    /C=US/ST=UT/L=Salt Lake City/O=The USERTRUST Network/OU=http://www.usertrust.com/CN=UTN-USERFirst-Hardware
    /C=US/ST=UT/L=Salt Lake City/O=The USERTRUST Network/OU=http://www.usertrust.com/CN=UTN-USERFirst-Object
    /C=EU/O=AC Camerfirma SA CIF A82743287/OU=http://www.chambersign.org/CN=Chambers of Commerce Root
    /C=EU/O=AC Camerfirma SA CIF A82743287/OU=http://www.chambersign.org/CN=Global Chambersign Root
    /C=HU/L=Budapest/O=NetLock Halozatbiztonsagi Kft./OU=Tanusitvanykiadok/CN=NetLock Minositett Kozjegyzoi (Class QA) Tanusitvanykiado/emailAddress=info@netlock.hu
    /C=HU/ST=Hungary/L=Budapest/O=NetLock Halozatbiztonsagi Kft./OU=Tanusitvanykiadok/CN=NetLock Kozjegyzoi (Class A) Tanusitvanykiado
    /C=HU/L=Budapest/O=NetLock Halozatbiztonsagi Kft./OU=Tanusitvanykiadok/CN=NetLock Uzleti (Class B) Tanusitvanykiado
    /C=HU/L=Budapest/O=NetLock Halozatbiztonsagi Kft./OU=Tanusitvanykiadok/CN=NetLock Expressz (Class C) Tanusitvanykiado
    /C=US/OU=www.xrampsecurity.com/O=XRamp Security Services Inc/CN=XRamp Global Certification Authority
    /C=US/O=The Go Daddy Group, Inc./OU=Go Daddy Class 2 Certification Authority
    /C=US/O=Starfield Technologies, Inc./OU=Starfield Class 2 Certification Authority
    /C=IL/ST=Israel/L=Eilat/O=StartCom Ltd./OU=CA Authority Dep./CN=Free SSL Certification Authority/emailAddress=admin@startcom.org
    /C=IL/O=StartCom Ltd./OU=Secure Digital Certificate Signing/CN=StartCom Certification Authority
    /C=TW/O=Government Root Certification Authority
    /C=ES/L=C/ Muntaner 244 Barcelona/CN=Autoridad de Certificacion Firmaprofesional CIF A62634068/emailAddress=ca@firmaprofesional.com
    /C=US/O=Wells Fargo/OU=Wells Fargo Certification Authority/CN=Wells Fargo Root Certificate Authority
    /C=ch/O=Swisscom/OU=Digital Certificate Services/CN=Swisscom Root CA 1
    /C=US/O=DigiCert Inc/OU=www.digicert.com/CN=DigiCert Assured ID Root CA
    /C=US/O=DigiCert Inc/OU=www.digicert.com/CN=DigiCert Global Root CA
    /C=US/O=DigiCert Inc/OU=www.digicert.com/CN=DigiCert High Assurance EV Root CA
    /C=FR/O=Certplus/CN=Class 2 Primary CA
    /O=Digital Signature Trust Co./CN=DST Root CA X3
    /C=US/O=Digital Signature Trust/OU=DST ACES/CN=DST ACES CA X6
    /CN=T\xC3\x9CRKTRUST Elektronik Sertifika Hizmet Sa\xC4\x9Flay\xC4\xB1c\xC4\xB1s\xC4\xB1/C=TR/L=ANKARA/O=(c) 2005 T\xC3\x9CRKTRUST Bilgi \xC4\xB0leti\xC5\x9Fim ve Bili\xC5\x9Fim G\xC3\xBCvenli\xC4\x9Fi Hizmetleri A.\xC5\x9E.
    /CN=T\xC3\x9CRKTRUST Elektronik Sertifika Hizmet Sa\xC4\x9Flay\xC4\xB1c\xC4\xB1s\xC4\xB1/C=TR/L=Ankara/O=T\xC3\x9CRKTRUST Bilgi \xC4\xB0leti\xC5\x9Fim ve Bili\xC5\x9Fim G\xC3\xBCvenli\xC4\x9Fi Hizmetleri A.\xC5\x9E. (c) Kas\xC4\xB1m 2005
    /C=CH/O=SwissSign AG/CN=SwissSign Platinum CA - G2
    /C=CH/O=SwissSign AG/CN=SwissSign Gold CA - G2
    /C=CH/O=SwissSign AG/CN=SwissSign Silver CA - G2
    /C=US/O=GeoTrust Inc./CN=GeoTrust Primary Certification Authority
    /C=US/O=thawte, Inc./OU=Certification Services Division/OU=(c) 2006 thawte, Inc. - For authorized use only/CN=thawte Primary Root CA
    /C=US/O=VeriSign, Inc./OU=VeriSign Trust Network/OU=(c) 2006 VeriSign, Inc. - For authorized use only/CN=VeriSign Class 3 Public Primary Certification Authority - G5
    /C=US/O=SecureTrust Corporation/CN=SecureTrust CA
    /C=US/O=SecureTrust Corporation/CN=Secure Global CA
    /C=GB/ST=Greater Manchester/L=Salford/O=COMODO CA Limited/CN=COMODO Certification Authority
    /C=NL/O=DigiNotar/CN=DigiNotar Root CA/emailAddress=info@diginotar.nl
    /C=US/O=Network Solutions L.L.C./CN=Network Solutions Certificate Authority
    /C=US/O=Wells Fargo WellsSecure/OU=Wells Fargo Bank NA/CN=WellsSecure Public Root Certificate Authority
    ---
    SSL handshake has read 19414 bytes and written 406 bytes
    ---
    New, TLSv1/SSLv3, Cipher is DHE-RSA-AES256-SHA
    Server public key is 2048 bit
    Secure Renegotiation IS NOT supported
    Compression: NONE
    Expansion: NONE
    SSL-Session:
        Protocol : TLSv1
        Cipher : DHE-RSA-AES256-SHA
        Session-ID: 5E47469313749129FE45EB7B860A89736CEDCB0201150177B44539192C192A6A
        Session-ID-ctx:
        Master-Key: 389653DA98920D0E1558EAF600286255BD582ADAADD188607A14AA15997279F30C7CAC3B8DFB322D2E7A378DEF83EEBB
        Key-Arg : None
        PSK identity: None
        PSK identity hint: None
        SRP username: None
        TLS session ticket:
        0000 - cb 0a 97 a7 b8 56 2d c7-d2 72 3f 2d f0 4d 0c 60 .....V-..r?-.M.`
        0010 - 92 1b a7 11 a8 41 56 14-47 f0 a3 15 52 f8 92 d6 .....AV.G...R...
        0020 - 48 58 26 89 f0 a4 a5 53-8e 43 8c 0e e0 6c 05 c9 HX&....S.C...l..
        0030 - 16 b9 2c 55 b4 2e e2 99-f7 5c 56 95 85 8a 1d ed ..,U.....\V.....
        0040 - 95 07 b3 a4 4d 2b a6 78-c1 da f4 e7 32 84 ce b5 ....M+.x....2...
        0050 - 11 f8 3c f5 1b 3e e6 07-dd 2d dc 2c b8 31 ba ad ..<..>...-.,.1..
        0060 - f5 e3 f1 52 6c b5 e9 79-87 8a 49 2c d4 52 2e b2 ...Rl..y..I,.R..
        0070 - 8e b7 e1 fe 8d 0a a5 63-28 ac f3 23 4a 02 d2 56 .......c(..#J..V
        0080 - af f5 c0 77 3a b7 b3 38-9f cc ac c5 78 56 b8 ce ...w:..8....xV..
        0090 - 8e b6 c8 2d 96 d9 41 fc-06 a3 29 a3 c0 cc 4d ...-..A...)...M
        00a0 - <SPACES/NULS>
    
        Start Time: 1396997911
        Timeout : 300 (sec)
        Verify return code: 21 (unable to verify the first certificate)
    ---
    250 HELP
    

The real clue was that `s_client` was only showing one certificate in the chain:

    ---
    Certificate chain
     0 s:/OU=Domain Control Validated/OU=PositiveSSL/CN=mail.hero.com
       i:/C=GB/ST=Greater Manchester/L=Salford/O=COMODO CA Limited/CN=PositiveSSL CA 2
    ---
    

Why??? Why, dear God, why?

The answer was [here][2]

For some reason, I suspect sendmail loops through each of the certificates listed in the pem and tries to validate them through the OpenSSL library. That library relies upon the aforementioned `ca-bundle.crt`. So, the answer was to cat the Comodo-supplied intermediate and root certifcates into `ca-bundle.crt`.

    $ openssl x509 -in AddTrustExternalCARoot.crt -text -noout >>  ca-bundle.crt
    $ cat AddTrustExternalCARoot.crt >> ca-bundle.crtt
    $ openssl x509 -in PositiveSSLCA2.crt -text -noout >> ca-bundle.crt
    $ cat PositiveSSLCA2.crt >> ca-bundle.crt
    

Voila!

 [1]: http://www.digicert.com/ssl-support/pem-ssl-creation.htm
 [2]: https://groups.google.com/forum/#!topic/comp.mail.sendmail/fkMqNUxiL9Q