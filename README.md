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

```
tinygo flash -target=wioterminal -port /dev/ttyACM0 [le chemin de votre programme]
```

- Testons 2 programmes blinky2.go et echo.go

```
TEST 1
mamadou@dugny:~$ cat /usr/local/lib/tinygo/src/examples/blinky2/blinky2.go
package main
// This blinky is a bit more advanced than blink1, with two goroutines running
// at the same time and blinking a different LED. The delay of led2 is slightly
// less than half of led1, which would be hard to do without some sort of
// concurrency.
import (
        "machine"
        "time"
)

func main() {
        go led1()
        led2()
}

func led1() {
        led := machine.LED1
        led.Configure(machine.PinConfig{Mode: machine.PinOutput})
        for {
              println("+")
              led.Low()
              time.Sleep(time.Millisecond * 1000)
        }  
}
println("-")
led.High()
time.Sleep(time.Millisecond * 1000)
func led2() {
          led := machine.LED2
          led.Configure(machine.PinConfig{Mode: machine.PinOutput})
          for {
                    println(" +")
                    led.Low()
                    time.Sleep(time.Millisecond * 420)
                    println(" -")
                    led.High()
                    time.Sleep(time.Millisecond * 420)
          }
}
mamadou@dugny:~$ tinygo flash -target wioterminal -port /dev/ttyACM0
/usr/local/lib/tinygo/src/examples/blinky2/blinky2.go
# command-line-arguments
/usr/local/lib/tinygo/src/examples/blinky2/blinky2.go:19:17: LED1 not declared
by package machine
/usr/local/lib/tinygo/src/examples/blinky2/blinky2.go:33:17: LED2 not declared
by package machine

TEST 2
mamadou@dugny:~$ cat /usr/local/lib/tinygo/src/examples/echo/echo.go
// This is a echo console running on the device UART.
// Connect using default baudrate for this hardware, 8-N-1 with your terminal
program.
package main
import (
                "machine"
                "time"
)
// change these to test a different UART or pins if available
var (
                uart = machine.Serial
                tx = machine.UART_TX_PIN
                rx = machine.UART_RX_PIN
)
func main() {
                uart.Configure(machine.UARTConfig{TX: tx, RX: rx})
                uart.Write([]byte("Echo console enabled. Type something then press
                enter:\r\n"))
                input := make([]byte, 64)
                i := 0
                for {
                                if uart.Buffered() > 0 {
                                data, _ := uart.ReadByte()
                                switch data {
                                                case 13:
                                                // return key
                                                uart.Write([]byte("\r\n"))
                                                uart.Write([]byte("You typed: "))
                                                uart.Write(input[:i])
                                                uart.Write([]byte("\r\n"))
                                                i = 0
                                                default:
                                                // just echo the character
                                                uart.WriteByte(data)
                                                input[i] = data
                                                i++
                                }
                      }
                }
}
time.Sleep(10 * time.Millisecond)
mamadou@dugny:~$ tinygo flash -target wioterminal -port /dev/ttyACM0
/usr/local/lib/tinygo/src/examples/echo/echo.go

``` 

