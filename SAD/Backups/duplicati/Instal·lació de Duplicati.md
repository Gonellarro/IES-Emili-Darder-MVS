![ImatgeCaplçaelra](attachments/duplicati.png)
# Instalació de duplicati a Ubuntu 24.04

Per tal de tenir els nostres sistemes segurs, hem de fer còpies de seguretat de forma segura, planificada, eficient i emprant els serveis estàndards d'avui dia. És per això que hem triat l'eina duplicati, ja que ens permet de dur a terme el que comentam aquí.

---
**APRENDRÀS:**

- **wget i apt install**: Fem feina amb aquestes eines per instal·lar duplicati
- **systemctl**: Empram el systemctl per habilitar el servei
- **Instal·lar Duplicati**: instal·lam i configuram el servidor

---
## Instal·lació en local

Recomano anar abans a la [pàgina de descàrregues de duplicati](https://duplicati.com/download) i copiar l'enllaç del fitxer actual de linux/ubuntu/debian.

```
wget https://updates.duplicati.com/beta/duplicati-2.1.0.3_beta_2025-01-22-linux-x64-gui.deb
sudo apt install ./duplicati-2.1.0.3_beta_2025-01-22-linux-x64-gui.deb
sudo nano /etc/default/duplicati
```

Insertam el següent codi a l'arxiu de configuració (/etc/default/duplicati):

```bash
sudo nano /etc/default/duplicati/
```

```
DAEMON_OPTS="--webservice-interface=any --webservice-port=8200 --webservice-password=contrasenyaSecretaWEBGUI"
```

Aquesta contrasenya fa referència a la contrasenya del WebGUI. Si no la posem, es genera una aletoria, que no coneixem.

Iniciam el servei i el deixam en marxa:

```
systemctl start duplicati
systemctl enable duplicati
```

Ja podem accedir a duplicati des del web: http://IP-SERVIDOR:8200

## Instal·lació mitjançant docker compose

Per instal·lar duplicati mitjançant docker compose, emprarem el que hi ha a [linuxserver.io](https://docs.linuxserver.io/images/docker-duplicati/#version-tags), que és el següent fitxer de docker-compose.yml:

```yaml
services:
  duplicati:
    image: lscr.io/linuxserver/duplicati:latest
    container_name: duplicati
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Madrid
      - SETTINGS_ENCRYPTION_KEY=clauSecreta
      - CLI_ARGS= #optional
      - DUPLICATI__WEBSERVICE_PASSWORD= #optional
    volumes:
      - /home/marti/docker/duplicati/config:/config
      - /mnt/backup:/backup
      - /home/marti:/source
      - /home/marti/docker/duplicati/scripts:/scripts # opcional
      - /var/run/docker.sock:/var/run/docker.sock # opcional
    ports:
      - 8200:8200
    restart: unless-stopped
```

Cream l'estructura de directoris, guardam dins el docker-compose.yml i ho llançam mitjançant la comanda:

```
docker compose up -d
```

>⚠️ **Hem de dir que**: emprar docker compose malgrat a primera vista sigui parescut, relament no ho és. Hem de tenir present, que probablement haguem d'aturar altres serveis, contenidors o bases dades abans de fer les còpies de seguretat. Això fa que s'hagi de configurar molt més per tal de que el contenidor de duplicati pugui realitzar aquestes tasques, que si es fa mitjançant el servei propi del sistema, no són necessari.


