![ImatgeCapçalera](attachments/windowsFirefox.png)
# Instal·lació del certificat arrel
---
Per a què la CA sigui reconeguda pel nostre ordinador, s’ha d’instal·lar el certificat arrel al magatzem de certificats del SO o a les aplicacions que vulguem que els emprin. 

En aquesta pràctica suposarem que els dispositius més comuns d’escriptori són Windows i instal·larem el certificat al magatzem de certificats de Windows i alternativament, també ho farem per al navegador Firefox, que no cull els certificats del sistema operatiu, sino que té el seu propi magatzem de certificats.

---
**APRENDRÀS:**

- **Magatzem de certificats de Windows**: Aprendràs on està el magatzem de certificats de Windows i com accedir-hi
- **Gestió de certificats a Firefox**: Aprendràs on està el magatzem de certificats de Firefox i com accedir-hi
- **Importar certificats**: Importar els certificats arrel de la nostra AC
---
## Windows

Per això feim: 

- Tecla Windows  
- Escrivim: certlm.msc

Ens apareix l’mmc del magatem de certificats. Anam a “**Entidades de certificación raíz de confianza/Certificados**” pulsam botó dret i triam “**Todas las tareas/Importar**”. Sel·leccionam el fitxer myCA.pem (abans l’hem de descarregar del servidor i copiar-lo al PC amb Windows). D’aquesta forma aquest PC ja confia amb la nostra nova CA.


## Firefox

Feim click a les tres retxes de dalt a la dreta per obrir el menú i sel·leccionam “**Ajustes**”.  Sel·leccionam “**Privacidad y Seguridad**”. Aquí baixam fins a trobar “**Certificados**” i picam damunt “**Ver certificados**”. Finalment picam damunt "**Importar**", tenguent present que hem de tenir sel·lcionades les Autoritats de certificació.
