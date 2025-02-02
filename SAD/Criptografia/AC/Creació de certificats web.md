![ImatgeCapçalera](attachments/ssl.png)
# Creació de certificats web

En aquesta documentació suposarem que ja hem creat una AC i podem signar certificats per a qualsevol nou web que necessiti HTTPS. 

Aquí veurem:
- com es creen les claus i el certificat del lloc web
- com fer una sol·licitud al servidor de la AC per a què ens signi el certificat
- assignar els rols que volem al certificat

---
**APRENDRÀS:**

- **A**: AAAA
- **B**: BBBB
- **C**: CCCC
---
## Creació del certificat i les claus

Primer, cream ls parella de claus pública/privada per al lloc de web i n’extreim la clau privada. Tingueu en compte que anomenem la clau privada mitjançant l'URL del nom de domini del lloc web (assumim que el loc es diu **ubntsrv02.local**). Això no és estrictament necessari, però facilita la gestió si tenim més llocs web:

```bash
openssl genrsa -out ubntsrv02.local.key 2048
```

## Creació de la sol·licitud de signat

Ara hem de demanar a l'AC que ens signi el certificat del lloc amb la clau privada de l'AC. Per això es fa una sol·licitud anomenada CSR:

```bash
openssl req -new -key ubntsrv02.local.key -out ubntsrv02.local.csr
```

Ens farà les mateixes preguntes que abans però en aquest cas, les nostres respostes fan referència al servidor al que volem crear el certificat.

```bash
marti@ubntsrv1:~/certs$ openssl req -new -key ubntsrv02.local.key -out ubntsrv02.local.csr  
You are about to be asked to enter information that will be incorporated  
into your certificate request.  
What you are about to enter is what is called a Distinguished Name or a DN.  
There are quite a few fields but you can leave some blank  
For some fields there will be a default value,  
If you enter '.', the field will be left blank.  
-----  
Country Name (2 letter code) [AU]:SP  
State or Province Name (full name) [Some-State]:Illes Balears  
Locality Name (eg, city) []:Palma de Mallorca  
Organization Name (eg, company) [Internet Widgits Pty Ltd]:IES Emili Darder  
Organizational Unit Name (eg, section) []:Dept TIC  
Common Name (e.g. server FQDN or YOUR name) []:ubntsrv02.local  
Email Address []:  
  
Please enter the following 'extra' attributes  
to be sent with your certificate request  
A challenge password []:  
An optional company name []:  
marti@ubntsrv1:~/certs$
```

La “challenge password” és bàsicament un secret compartit entre noltros  i l'emissor del certificat SSL (l’autoritat de certificació o AC), incrustat a la CSR, que l'emissor (AC) pot utilitzar per autenticar-mos si alguna vegada ho necessiteu. **No** serveix per encriptar més o menys. Si no posam la challenge password, la clau nova estarà igualment encriptada.

>**Nota:** Important triar bé el *Common Name*, ja que serà una part cabdal a l’hora de publicar el certificat

## Assignació els rols al certificat

Finalment, crearem un fitxer de configuració d'extensió de certificat X509 V3, que s'utilitza per definir el nom alternatiu del subjecte (SAN) per al certificat. En el nostre cas, crearem un fitxer de configuració anomenat ubntsrv02.local.ext que conté el text següent i el guardarem allà on estem:

```bash
authorityKeyIdentifier=keyid,issuer  
basicConstraints=CA:FALSE  
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment  
subjectAltName = @alt_names  
  
[alt_names]  
DNS.1 = ubntsrv02.local
```

Executarem openssl x509 perquè l'ordre x509 ens permet editar la configuració de confiança del certificat. En aquest cas, l'estem utilitzant per signar el certificat juntament amb el fitxer de configuració, que ens permet establir el Nom alternatiu del subjecte (SAN). 

Ara executem l'ordre per crear el certificat: utilitzant el nostre CSR, la clau privada de l'AC, el certificat de l'AC i el fitxer de configuració:

```bash
openssl x509 -req -in ubntsrv02.local.csr -CA myCA.pem -CAkey myCA.key -CAcreateserial -out ubntsrv02.local.crt -days 825 -sha256 -extfile ubntsrv02.local.ext
```

Obtenim un resultat com aquest:

```bash
marti@ubntsrv1:~/certs$ openssl x509 -req -in ubntsrv02.local.csr -CA myCA.pem -CAkey myCA.key -CAcreateserial -out ubntsrv02.local.crt -days 825 -sha256 -extfile ubntsrv02.local.ext  
Signature ok  
subject=C = SP, ST = Illes Balears, L = Palma de Mallorca, O = IES Emili Darder, OU = Dept TIC, CN = ubntsrv02.local  
Getting CA Private Key  
Enter pass phrase for myCA.key:  
marti@ubntsrv1:~/certs$
```

>**Un detall:** Quan ens demana la contrasenya, és la contrasenya per accedir **a la clau privada de l'AC**. És important tenir aquest concepte clar, ja que cada pic que estam emprant la clau privada de l'AC, haurem de proporcionar la seva contrasenya.

Si revisam ara els fitxers que tenim, veim que hauríem de tenir els següents:

```bash
marti@ubntsrv1:~/certs$ ls -l  
total 32  
-rw------- 1 marti marti 1751 may  5 13:19 myCA.key  
-rw-rw-r-- 1 marti marti 1415 may  5 13:21 myCA.pem  
-rw-rw-r-- 1 marti marti  451 may  5 13:21 myCApubkey.pem  
-rw-rw-r-- 1 marti marti   41 may  5 13:51 myCA.srl  
-rw-rw-r-- 1 marti marti 1432 may  5 13:51 ubntsrv02.local.crt  
-rw-rw-r-- 1 marti marti 1050 may  5 13:45 ubntsrv02.local.csr  
-rw-rw-r-- 1 marti marti  207 may  5 13:51 ubntsrv02.local.ext  
-rw------- 1 marti marti 1675 may  5 13:39 ubntsrv02.local.key
```

Els fitxers resultants són: 
- ubntsrv02.local.key (la clau privada)
- ubntsrv02.local.csr (la sol·licitud de signatura del certificat o fitxer csr)
- ubntsrv02.local.crt (el certificat signat). 

Ara ja podem configurar servidors web locals per utilitzar HTTPS amb la clau privada i el certificat signat.