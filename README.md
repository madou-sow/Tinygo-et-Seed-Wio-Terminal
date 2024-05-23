## Tinygo et Seed Wio Terminal
Le terminal Seeed Wio est une petite carte de développement ARM basée sur la famille de SoC
Atmel ATSAMD51P20

### Flashing
#### UF2
Le terminal Wio est livré avec le bootloader UF2 déjà installé, Format de flash USB (UF2). UF2 est
un format de fichier, développé par Microsoft pour PXT (également connu sous le nom de
Microsoft MakeCode), qui est particulièrement adapté pour flasher les microcontrôleurs sur MSC
(Mass Storage Class ; alias lecteur flash amovible).
Le fichier UF2 se compose de blocs de 512 octets, chacun étant autonome et indépendant des
autres. Chaque bloc de 512 octets se compose de (voir ci-dessous pour plus de détails) :

- nombres magiques au début et à la fin

- adresse où les données doivent être flashées

- jusqu'à 476 octets de données

Les transferts de données via MSC arrivent toujours par multiples de 512 octets. Avec la structure
du système de fichiers FAT, cela signifie que les blocs du fichier UF2 sont toujours alignés avec les
écritures MSC - le microcontrôleur ne reçoit jamais de fichier partiel.

Les nombres magiques permettent au microcontrôleur de distinguer un bloc de fichier UF2 d'autres
données (par exemple, une entrée de table FAT ou divers fichiers de comptabilité stockés par
certains systèmes d'exploitation). Lorsqu'un bloc UF2 est reconnu, il peut être flashé
immédiatement (sauf si la taille de la page flash est supérieure à 256 octets ; dans ce cas, un tampon
est nécessaire). La gestion réelle du format de fichier lors de l'écriture est très simple (~ 10 lignes de
code C dans la version la plus simple).

Flashing UF2 pour Linux : Ce référentiel contient des scripts et des correctifs pour créer un exemple
d'image Linux basée sur piCore pour le Raspberry Pi Zero. L'image est censée démarrer très
rapidement (actuellement autour de 7 secondes) et exposer un périphérique de stockage de masse
USB (clé USB), qui peut être utilisé pour programmer un Raspberry Pi Zero avec des fichiers UF2,
généralement générés à partir de Microsoft MakeCode et en particulier de MarqueCode Arcade.
L'image a été testée sur un Raspberry Pi Zero Rev 1.3 et Zero W Rev 1.3. Il pourrait théoriquement
fonctionner sur le Pi A/A+ d'origine, mais n'a pas été testé. D'autres modèles n'ont pas la broche
d'identification OTG et ne peuvent donc pas être utilisés en mode périphérique USB.

### CLI (Command-Line Interface) Clignotant
Une interface en ligne de commande ou ILC (traduction de CLI) est une interface homme-machine
dans laquelle la communication entre l'utilisateur et l'ordinateur s'effectue en mode texte :
l'utilisateur tape une ligne de commande, c'est-à-dire du texte au clavier pour demander à
l'ordinateur d'effectuer une opération.

- Branchez votre Wio Terminal sur le port USB de votre ordinateur.
- les différents exemples préinstallés avec le package tinygo_0.26.0_amd64.deb

```
mamadou@dugny:/usr/local/lib/tinygo/src/examples$ ls -1
adc
blinkm
blinky1
blinky2
button
button2
can
caninterrupt
dac
echo
echo2
gba-display
hid-keyboard
hid-mouse
i2s
mcp3008
memstats
microbit-blink
pininterrupt
pwm
rand
serial
systick
temp
test
uart
usb-midi
wasm
```

- Flashez votre programme TinyGo sur la carte en utilisant cette commande :

``
tinygo flash -target=wioterminal -port /dev/ttyACM0 [le chemin de votre programme]
```


