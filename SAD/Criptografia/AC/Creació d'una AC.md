![ImatgeCapçalera](attachments/ac.png)
# Creació d'una AC en Ubuntu Server
---
En aquesta documentació es pretén crear una Autoritat de Certificació (en anglés Certification Authority o CA), configurar el certificat arrel als dispositius clients i extreure les claus públiques i privades del certificat arrel.

El procés està orientat a saber com es crea de la forma més senzilla i com implementar-lo en tot el procés en vistes de poder generar certificats digitals tant a persones com a servidors web.

---
**APRENDRÀS:**

- **Què és una AC**: Entendre que és una Autoritat de Certificació capaç d'expedir certificats digitals.
- **OpenSSL**: La comanda que permet crear les claus asimètriques, certificats, etc...
- **Crear certificats arrel**: El certificat que ha de tenir la AC per poder expedir i signar els certificats digitals a qui els demani
- **Extreure les claus públiques i privades**: Obtenir les claus pública i privada del certificat arrel.
---
# Creació de l' AC

Primerament hem de triar quin sistema operatiu emprarem per muntar l'AC. En aquest cas hem fet servir un servidor Ubuntu 22.04 LTS. Es pot emprar qualsevol altre, tant windows com mac, però per aquesta activitat, hem triat Linux i Ubuntu per la facilitat d’implementació que té i cost nul.

Ara hem d’entendre què és una Autoritat de Certificació. L’autoritat de certificació  és l’encarregada de crear certificats digitals i signar-los per tal de que tothom que tengui com a Autoritat de Certificació d’arrel de confiança a aquesta en el seu navegador o magatzem de certificats, confiï en tot aquell que tingui un certificat signat per aquesta. En poques paraules, la CA és una mena de policia que expedeix DNI’s vàlids i en la que tots confiem per tal de saber que una persona/web és qui diu esser.

Un cop entès que tenim per una banda una autoritat en la que confiar i per altra que signa, vegem com es du a terme la creació d’aquesta autoritat.

Per aquesta activitat farem una CA molt senzilla i que s’encarregui només de signar certificats a pàgines web. Si volguéssim que tingués més funcions, hauríem d’ampliar la CA. Però pel que volem inicialment, amb el que proposarem basta.

És realment fàcil crear la CA. Tan sols hem de generar els fitxers necessaris per convertir-se en una autoritat de certificació i es resumeix en només dues ordres.

# Generació de la clau privada i el certificat arrel de l'AC


En la versió d’ubuntu server 22.04 tenim disponible *openssl*, aplicació amb la qual podem generar aquestes claus. Si empram una altra distribució i no està instal·lada, primer hauríem de descarregar-la habitualment mitjançant el gestor de paquets de la distribució.

Cream una carpeta on guardar els certificats:

```bash
mkdir ~/certs  
cd ~/certs
```

Cream ara la parella de clau pública/privada de la CA, però li demanam que ens doni només la clau privada. Punts a tenir en compte:

- Empram un algoritme de clau asimètrica. En aquest cas és rsa, ja que indicam *genrsa.* Si consultam [l’ajuda de OpenSSL](https://www.openssl.org/docs/manmaster/man1/) veim que tenim les possibilitats d’emprar RSA, DSA o emprar un algoritme que li indiquem noltros. En aquest cas, hem triat RSA com a algoritme de clau asímètrica i li extraurem la clau privada.  
- Per tal de que si la perdem no puguin tenir accés a la clau privada, l’encriptam amb un algoritme de clau simètrica, en aquest cas 3DES. Si consultam el [manual d’OpenSSL](https://www.openssl.org/docs/manmaster/man1/openssl-genrsa.html) veurem que hi ha diferents algoritmes de clau simètrica per encriptar la clau privada (aes, aria, camelia, DES, 3DES, Idea). Això vol dir que ens demanarà una contrasenya per poder encriptar-la.   
- Finalment li indicam quin és el fitxer on es guardarà la clau privada

```bash
openssl genrsa -des3 -out myCA.key 2048
```

El procés crea un fitxer que es diu myCA.key que conté la clau privada, encriptada amb 3DES i amb una longitud en bits de la clau privada de 2048\.

El procés ens dóna una sortida com la següent:

```bash
marti@ubntsrv1:~/certs$ openssl genrsa -des3 -out myCAcripted.key 2048  
Generating RSA private key, 2048 bit long modulus (2 primes)  
................+++++  
.....+++++  
e is 65537 (0x010001)  
Enter pass phrase for myCAcripted.key:  
Verifying - Enter pass phrase for myCAcripted.key:  
marti@ubntsrv1:~/certs$
```

Respecte als símbols que ens surten quan es crea la clau privada, [el manual](https://www.openssl.org/docs/manmaster/man1/openssl-genrsa.html) ens explica que signifiquen:

> “RSA private key generation essentially involves the generation of two or more prime numbers. When generating a private key various symbols will be output to indicate the progress of the generation. A . represents each number which has passed an initial sieve test, + means a number has passed a single round of the Miller-Rabin primality test, * means the current prime starts a regenerating progress due to some failed tests. A newline means that the number has passed all the prime tests (the actual number depends on the key size).
> 
> Because key generation is a random process the time taken to generate a key may vary somewhat. But in general, more primes lead to less generation time of a key.”

Ara el que hem de crear és el certificat arrel (root certificate), que no és res més de signar el propi certificat de la CA.

Ara el que hem de crear és el certificat arrel (root certificate), que no és res més de signar el propi certificat de la CA.

```bash
openssl req -x509 -new -key myCA.key -sha256 -days 1825 -out myCA.pem
```

Amb aquesta comanda el que feim és el següent:

- req: crea i processa sol·licituds de certificat (CSR) en format PKCS\#10. A més, pot crear **certificats autosignats** per utilitzar-los com a CA arrel. Veure a la [documentació](https://www.openssl.org/docs/man1.0.2/man1/openssl-req.html).  
- [x509:](https://es.wikipedia.org/wiki/X.509) genera un certificat en comptes de una petició de certificat (CSR)  
- new: genera una nova sol·licitud de certificat. **Sol·licita a l'usuari els valors de camp rellevants.**  
- key: indica amb quina clau privada hem d’autosignar el certificat arrel.  
- sha256: empra l’algoritme sha256 per fer el hash del certificat.  
- days: indica els dies que ha d’estar actiu. Més enllà d’aquest dia, el certificat deixa de ser vàlid.

El procés ens dóna una sortida com la següent:

```bash
marti@ubntsrv1:~/certs$ openssl req -x509 -new -key myCA.key -sha256 -days 1825 -out myCA.pem  
Enter pass phrase for myCA.key:  
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
Common Name (e.g. server FQDN or YOUR name) []:ubntsrv01  
Email Address []:  
marti@ubntsrv1:~/certs$
```

En aquest cas, ens demana la clau amb la que hem encriptat la clau privada, ja que hi ha d’accedir a ella per crear el certificat arrel. 

Indicam les dades del certificat: està a Espanya, a les Illes Balears, Palma de Mallorca, l’organització és l’IES Emili Darder, el departament és el de TIC i el servidor des d’on s’està creant és el ubntsrv01. Deixam l’adreça de correu en blanc, ja que no hi ha cap responsable ara com ara d’aquest certificat de proves.

Ara ja hauríem de tenir 2 fitxers: el fitxer de la clau privada (myCA.key) i el certificat arrel (myCA.pem).

Si editam la clau privada, podem veure que és algo així:

```bash
-----BEGIN RSA PRIVATE KEY-----  
Proc-Type: 4,ENCRYPTED  
DEK-Info: DES-EDE3-CBC,E72C1E12AB0C95BF  
  
mCzqSOnrHDsUJpHGOoXn3Yw3HAvSkKk/z+psbaTpB7gwASW+V01Fq6Wd/X+SQq9M  
XIJ7oNpoC6CBN8Ykn+loa5+W8dby4fiLKGOCi3+ouawxyMqZxHV/PsTNNvU/lXUX  
JUwyADSKptS3vu8Xspu79xohuv1t/AWV+3wEtUuCFL5d0vt/kb+kLpuLArDERy/a  
GdQUktymdtBq/HO4TnAh3CsVOK/L6DatnwC9hgZjrOQze4Yt8mWnoTR3yh6t6Vro  
iFp0+Nmn5xV/xrYCwjpiAMhe0epe7YCi3pYYYNuq5X1fl7qGFYaCZ4Jmoi92G48B  
+Y7HqCruieZxgKvrT+vBlP9o97OfUF9+JuIuSqC72DGzF2GR52QuFNXBVs/nXeFk  
…  
4rwwHVGSkyRX/b5F/LQQ9oGikHnXvTuvWqMsqNlpa/wCUKwY9mJpQeKQ2VltBURN  
CKmVsi0URPDabHpWx7QdQzsR8HewYMrRRT+/Eb1JeHdX9FW+W84F90Y2dpVJKVE3  
EyGvJr8yxkWjyZYkoeRuKrAHZ40kyZ9J+LdBXy284LA6w8KzYVLm+VHYYNNv5HPi  
f6MYa6mQsN3KEUcJGSKxhUOhlFcvzDYLfE65hgtRvO3ec90akHX2Qw==  
-----END RSA PRIVATE KEY-----
```

Podem veure que la clau privada està encriptada amb 3DES, tal i com indica a la capçalera del fitxer.

**Opcional:** No és necessari, però si recomanable tenir la clau pública. Per això podem executar la següent ordre:

```bash
openssl rsa -in myCA.key -outform PEM -pubout -out myCApubkey.pem
```

D’aquesta forma ens genera la clau pública en format pem al fitxer myCApubkey.pem.

Si editam la clau pública obtenim un resultat com el següent:

```bash
-----BEGIN PUBLIC KEY-----  
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA05jD8Wu5JPQNH5NtBkXX  
sV1k3IXrXuFu8ZIXie34ArBVLd0wJ2toFJJYp5duCRVSEf8LzenAoxb82t+8008D  
iFH0kQhSqgvucK7vgqera7PRCxXIhIIWNZ3p8B2IxtWEExzacamEz1lFz0Jtb2Hf  
UZvvh2sk96m/m6f0KtsNaHNHj5kq7DrCZklORtEiZk7fXO6ZAwkJ49Qm5JUYY+jf  
kLELy6PEatP2nHGh96EVjHbUUEcDS70dol8CL8FGVQbHD69mDlXjcWhrnkpq4u6O  
nBt6FYhzmfZ7ELwgDimcMUtTZdL7Q2nK7QhD2VijLageCsXoAogFQu/IhiH3OKuN  
sQIDAQAB  
-----END PUBLIC KEY-----
```
