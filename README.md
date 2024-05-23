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
- La carte Wio Terminal devrait redémarrer puis commencer à exécuter votre programme

### Exemples de programmes en TinyGo compilés (flash) sur la borne WIO T
#### Test avec le programme blinkblue.go avec le MCU arduino

```
1. Liste des modèles de microcontrôleurs arduino installés par tinygo

mamadou@dugny:/usr/local/lib/tinygo/targets$ ls -1 arduino*
arduino-mega1280.json
arduino-mega2560.json
arduino-mkr1000.json
arduino-mkrwifi1010.json
arduino-nano-new.json
arduino-nano.json
arduino-nano33.json
arduino-zero.json
arduino.json

2. Le programme blinkblue.go

mamadou@dugny:~/big-data/cerin10102022/apprentissage/tinygo$ cat blinkblue/blinkblue.go
package main
import (
"machine"
"time"
)

func main() {
                // On board LED is connected to GPIO 2
                //var led machine.Pin= machine.Pin(13)
                var led machine.Pin= machine.Pin(2)
                // Configure PIN as output
                led.Configure(machine.PinConfig{Mode: machine.PinOutput})
}
// Infinite main loop
for {
                // Turn LED off
                led.Low()
                // Wait for 1 second
                time.Sleep(time.Millisecond * 1000)
                // Turn LED on
                led.High()
                // Wait for 1 second
                time.Sleep(time.Millisecond * 1000)
}

3. Compilation

mamadou@dugny:~/big-data/cerin10102022/apprentissage/tinygo$ tinygo flash -
target arduino -port /dev/ttyACM0 blinkblue/blinkblue.go
avrdude: stk500_recv(): programmer is not responding
avrdude: stk500_getsync() attempt 1 of 10: not in sync: resp=0x00
avrdude: stk500_getsync() attempt 1 of 10: not in sync: resp=0x00
avrdude: stk500_recv(): programmer is not responding
avrdude: stk500_getsync() attempt 2 of 10: not in sync: resp=0x00
avrdude: stk500_recv(): programmer is not responding
avrdude: stk500_getsync() attempt 3 of 10: not in sync: resp=0x00

```
#### Test avec le programme blinkblue.go sur la borne Wio T

```
1. Liste des modèles de microcontrôleurs wioterminal installés par tinygo
mamadou@dugny:/usr/local/lib/tinygo/targets$ ls -1 wioterminal*
wioterminal.json

2. Compilation
mamadou@dugny:~/big-data/cerin10102022/apprentissage/tinygo$ tinygo flash -
target wioterminal -port /dev/ttyACM0 blinkblue/blinkblue.go
La compilation se déroule correctement
```

#### Test avec le programme gorotines sur la borne Wio T
```
Téléchargez à partir https://github.com/sago35/tinygo-examples.git

mamadou@dugny:/tempo$ cd tinygo-examples-wiot/
mamadou@dugny:/tempo/tinygo-examples-wiot$ ls -l
total 48
-rw-r--r-- 1 mamadou users 1043 Jan 5 15:22 LICENSE.txt
-rw-r--r-- 1 mamadou users 1838 Jan 5 15:22 Makefile
-rw-r--r-- 1 mamadou users 2385 Jan 5 15:22 README.md
drwxr-xr-x 2 mamadou users 4096 Jan 5 15:22 deviceid
-rw-r--r-- 1 mamadou users
209 Jan 5 15:22 go.mod
-rw-r--r-- 1 mamadou users 12633 Jan 5 15:22 go.sum
drwxr-xr-x 3 mamadou users 4096 Jan 5 15:22 pininterrupt
drwxr-xr-x 15 mamadou users 4096 Jan 5 15:22 wioterminal
drwxr-xr-x 5 mamadou users 4096 Jan 5 15:22 xiao-ble

mamadou@dugny:/tempo/tinygo-examples-wiot$ make
tinygo build -o /tmp/test.hex -size short -target pyportal ./pininterrupt
code data bss | flash ram
.
.
.
tinygo build -o /tmp/test.hex -size short -target pyportal ./pininterrupt
go: downloading golang.org/x/sys v0.0.0-20220829200755-d48e67d00261
tinygo build -o /tmp/test.hex -size short -target wioterminal ./wioterminal/goroutines
.
.
.

go: downloading tinygo.org/x/tinyfont v0.3.1-0.20220718115734-9b135c3a5561
go: downloading github.com/muka/go-bluetooth v0.0.0-20220830075246-0746e3a1ea53
go: downloading github.com/godbus/dbus/v5 v5.0.3
go: downloading github.com/sirupsen/logrus v1.9.0
go: downloading github.com/fatih/structs v1.1.0
tinygo build -o /tmp/test.hex --target xiao-ble --size short ./xiao-ble/ble-led-client

```
```
$cat main.go

package main
import (
"fmt"
"image/color"
"machine"
"time"
"tinygo.org/x/drivers/ili9341"
)
var (
led1 = machine.LED
display *ili9341.Device
)
var (
black = color.RGBA{0, 0, 0, 255}
gray = color.RGBA{32, 32, 32, 255}
white = color.RGBA{255, 255, 255, 255}
red = color.RGBA{255, 0, 0, 255}
blue = color.RGBA{0, 0, 255, 255}
green = color.RGBA{0, 255, 0, 255} )

func initialize() error {
machine.SPI3.Configure(machine.SPIConfig{
SCK: machine.LCD_SCK_PIN,
SDO: machine.LCD_SDO_PIN,
SDI: machine.LCD_SDI_PIN,
Frequency: 48000000,
})
backlight := machine.LCD_BACKLIGHT
backlight.Configure(machine.PinConfig{Mode: machine.PinOutput})
display = ili9341.NewSPI(
machine.SPI3, machine.LCD_DC,
machine.LCD_SS_PIN,
machine.LCD_RESET,
)
display.Configure(ili9341.Config{
Rotation: ili9341.Rotation270,
})
display.FillScreen(black)
backlight.High()
return nil
}
func errDisp(err error) {
for {
fmt.Printf("%s\r\n", err.Error())
time.Sleep(10 * time.Second)
}
}
func main() {
err := initialize()
if err != nil {
errDisp(err)
}
label1 := NewLabel(240, 0x12)
label2 := NewLabel(240, 0x12)
chCnt1 := make(chan uint32, 1)
chCnt2 := make(chan uint32, 1)
go timer77ms(chCnt1)
go timer500ms(chCnt2)
for {
select {
case cnt := <-chCnt1:
label1.SetText(fmt.Sprintf("timer77ms : %04X", cnt), white)
display.DrawRGBBitmap(0, 30, label1.buf, label1.w, label1.h)
case cnt := <-chCnt2:
label2.SetText(fmt.Sprintf("timer500ms: %04X", cnt), white)
display.DrawRGBBitmap(0, 50, label2.buf, label2.w, label2.h) }
time.Sleep(1 * time.Millisecond)
}
}
func timer77ms(ch chan<- uint32) {
cnt := uint32(0
for {
ch <- cnt
cnt++
time.Sleep(77 * time.Millisecond)
}
}
func timer500ms(ch chan<- uint32) {
cnt := uint32(0)
for {
ch <- cnt cnt++
time.Sleep(500 * time.Millisecond)
}
}

```

<img alt="WIOT" src="https://github.com/madou-sow/Tinygo-et-Seed-Wio-Terminal/blob/main/images/goroutine-tinygoflash.jpg" width=50% height=50%  title="WIOT"/>

Cet exemple montre ce qui suit, reçu via le canal. TinyGo utilise goroutine/canal pour simplifier le
processus asynchrone.

https://github.com/madou-sow/Tinygo-et-Seed-Wio-Terminal/blob/main/images/sample-tinygoflash.jpg
