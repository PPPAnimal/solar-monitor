# Wiring

Everything on one page: the wall layout, the battery connections, the monitor's own wiring with its shunt
sensor, and the optional power supply. The same content is published as a web page at
<https://claude.ai/artifact/Vx3LUqenznZZ3UZ892j9KL>.

- [Wall layout](#wall-layout)
- [Battery tops](#battery-tops)
- [Cable reach](#cable-reach)
- [Monitor and shunt sensor](#monitor-and-shunt-sensor) (the shunt is optional)
- [Power supply](#power-supply) (optional)
- [Check before building](#check-before-building)

## Wall layout

Batteries, controller and inverter stacked on the 24 in wall beside the door. Drawn to scale, heights in
inches off the floor.

![Front and side views of the wall](img/wall-front-and-side.svg)

The controller sits 2 in toward the corner from centre so both 16 in leads reach. The inverter negative runs
through the gap between the inverter and the controller, then down to the shunt.

## Battery tops

![Top view of the four batteries](img/battery-tops.svg)

Negatives are taken from B4 through the shunt and positives from B1, so current crosses the whole bank and
the four batteries share evenly. The shunt lies along the negative row, where touching another negative post
does no harm.

## Cable reach

| Cable | Run | Needs | Have | Result |
|---|---|---|---|---|
| Inverter negative, 2 AWG | Inverter to shunt on B4 | ~31 in | 36 in | Reaches, about 5 in spare |
| Inverter positive, 2 AWG with 150 A fuse | Inverter to fuse to B1 + | ~22 in | 36 in | About 14 in to take up as a loop |
| Controller positive | Controller to B1 + | ~12 in | 16 in | Reaches |
| Controller negative | Controller to shunt | ~10 in | 16 in | Reaches |
| PV, 10 AWG | Disconnect to controller | ~40 in | plenty | Reaches |

Lengths are worked out from the drawing, with an allowance for bends. Measure with a string before drilling.

## Monitor and shunt sensor

How the battery shunt, the INA226 sensor board, the buck converter and the ESP32 connect.

**The shunt is optional.** The monitor works from the charge controller alone, with just the buck converter
and the ESP32. Adding the shunt and its sensor board is what gives battery current, load and amp-hours in
and out. Without them, leave out the INA226 and the two sense wires; everything else here stays the same.

**Where the shunt goes.** One end carries every battery negative and nothing else. The other end carries
every other negative: the charge controller, the inverter and the buck converter's black wire. Anything
connected straight to a battery negative post goes round the shunt and is not measured.

![Monitor wiring: buck converter, ESP32, INA226 and shunt](img/shunt-wiring.svg)

One fused wire from battery positive feeds the buck converter and nothing else. The only connections to the
shunt are the two sense wires on its small screws and the buck's black wire on the system-side bolt. No wire
runs from the battery to the sensor board. Lines that cross are not joined.

| From | To | Wire |
|---|---|---|
| Battery + | 2 A fuse, then buck red | 18-20 AWG |
| Buck black | Shunt, system-side bolt | 18-20 AWG |
| Buck USB socket | ESP32 USB port | USB cable |
| ESP32 3V3 and GND | INA226 VCC and GND | Jumper wire |
| ESP32 GPIO 21 and 22 | INA226 SDA and SCL | Jumper wire |
| INA226 IN+ and IN− | The shunt's two sense screws | 22 AWG twisted pair |
| INA226 VBS and ALE | Nothing | None |

- **Leave VBS empty.** Do not run a wire from the battery to the sensor board. The first board, built from an
  earlier version of this drawing, had VBS on battery positive, and it burned: 12 V reached a sense pin
  beside it and shorted to battery negative through the sense wires. The pin only gave a second voltage
  reading, and the monitor uses the controller's voltage without it.
- **Check the board before it goes near the battery.** With a meter on ohms: IN+ to IN− must not read near
  zero once R100 is off, and no soldered pin may read near zero to its neighbour. Cover each joint.
- **Remove R100 first.** The INA226 board's own 0.1 ohm resistor has to come off before it will read the
  external shunt correctly.
- **Buck negative.** This assumes the buck's input black and USB ground are joined inside, which is normal
  for this type. Check with a meter. If they are not, add a wire from INA226 GND to the shunt's system-side
  bolt.
- **Sign.** Either sense wire can go on either screw. If charging reads negative, tick "Sense wires are the
  other way round" on the monitor's Setup page.
- **Pins.** GPIO 21 and 22 are the ESP32's usual I2C pins and the ones the firmware uses.
- **Shunt rating.** The INA226 reads up to about 82 mV, so check the rating stamped on the shunt covers the
  inverter's full draw. Enter the rating on the Setup page.

**What the monitor should show.** Setup, Sensors reads "Battery shunt: Working". At night with nothing
switched on, about −0.08 A: the controller, the ESP32, the buck converter and the sensor. In sun with no
loads, the controller's charge current less that same small draw. If it says "Not responding", check VCC,
GND, SDA and SCL at both ends; a dead or unpowered sensor left on the wires makes the bus read as noise.

## Power supply

**The power supply is optional.** Skip this section unless the board also runs an XY-series supply, such as one feeding LED lights. It connects the supply's
4-pin port to the ESP32 so the monitor can show its volts, amps and on/off state. The small WiFi board that
came with the supply is not used.

You need the 4-wire cable that came with the supply, two 1 kΩ resistors, one 2 kΩ resistor (or two more 1 kΩ
joined end to end) and a multimeter.

![Power supply port to ESP32](img/power-supply-wiring.svg)

The ESP32 plugs in where the WiFi board would have gone, using the same cable. Three wires connect: ground,
and the two that carry data. The 2 kΩ resistor goes from the GPIO16 side of the first 1 kΩ across to ground;
together they lower the supply's signal to a level the ESP32 can take.

| Supply pin | Goes to | Through |
|---|---|---|
| GND | ESP32 GND | Straight wire |
| TXD | ESP32 GPIO16 | 1 kΩ in line, then 2 kΩ from GPIO16 to GND |
| RXD | ESP32 GPIO17 | 1 kΩ in line |
| 5V | Nothing | Tape the bare end |

1. **Power everything down.** Disconnect the supply's input and the ESP32's power before touching any wire.
2. **Leave the supply's 5V wire off.** The ESP32 keeps running from its own converter. Two power sources
   joined together fight each other, and this way the monitor stays up when the lights' supply is off.
3. **Find the four pins by their printed labels.** The names are printed on the supply's board beside the
   port. Wire colours differ between cables.
4. **Measure the TXD pin.** With the cable in the supply but not the ESP32, power the supply and measure TXD
   against GND. About 5 V: wire it as drawn. About 3.3 V: leave the 2 kΩ resistor out and keep both 1 kΩ.
   Power down again.
5. **Make the three connections.** Use the port's GND pin for ground, not the supply's output negative
   terminal. Nothing works without the grounds joined.
6. **Check the main page.** Within a few seconds a Power supply card appears. Compare it with the supply's
   own screen.
7. **Send back the report.** On Setup, under Power supply, type in the volts and amps the supply's screen is
   showing, then press Save report.

If the card does not appear, power down and swap the TXD and RXD wires at the supply's end. Some boards label
them from the other side's point of view, and the two 1 kΩ resistors make a wrong guess harmless.

The supply itself is fed straight from the 12 V batteries on its input terminals, with a fuse in the positive
wire close to the battery. For now the monitor only reads from the supply.

## Check before building

- **Roof line.** The board is drawn to 55 in off the floor. Confirm the wall is clear to that height at the
  corner end.
- **Controller over the batteries.** Victron advises against this because of battery gassing. Sealed gel
  batteries only vent if overcharged, so the risk is small, but it is the cost of keeping the 16 in leads.
- **Inverter orientation.** Drawn with the long side level. Check the Energizer manual allows wall mounting
  that way.
- **Plug clearance.** There is about 2.5 in between the outlet end and the door frame, so cords will stick
  out toward the doorway.
- **Positive posts face the room.** Fit terminal covers, since those are the posts a dropped tool would
  reach.
- **Three lugs on B1 +.** The battery link, the inverter cable and the controller lead share that post. Put
  the 2 AWG lug against the post and use a bolt long enough for all three.
- **Estimated sizes.** The fuse holder, shunt, disconnect and electronics are drawn from rough sizes, and the
  platform is a suggestion.
- **40 A controller fuse.** When it is added, it goes in the controller positive lead, close to B1.
