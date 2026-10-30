# Gadnic Linux Driver — Community Fix

[Español](#español) · [English](#english)

A community-tested fork of the [VINSA 1060 Plus Linux driver by feveal](https://github.com/feveal/Tablet-VINSA-1060-Plus-Linux-Driver), with changes for a Gadnic drawing tablet using USB ID **`08f2:6811`** (reported as **T1161**).

**Tested environment:** Arch Linux, KDE Plasma (Wayland), and Krita. **Compatibility with other distributions or tablet models has not yet been verified.**

> **Important:** This is a community modification, not an official Gadnic driver. Keep a backup of any existing working configuration. Only run one instance of the driver at a time.

---

## Español

### ¿Qué mejora esta versión?

En el dispositivo probado:

- **Trazos rápidos más estables:** se eliminó el suavizado de coordenadas en **modo tableta**, evitando el retraso que provocaba ondulaciones. El modo mouse conserva su suavizado.
- **Menos trazos fantasma:** la normalización de presión considera `raw_pressure == 0` como ausencia de contacto.
- **Sensibilidad a la presión:** se mantiene el comportamiento de presión que funcionó en Krita durante las pruebas.
- **Mapeo absoluto:** se conserva el modo tableta que permite utilizar el área de dibujo.
- **Inicio automático opcional:** ejemplo de servicio `systemd` para iniciar el controlador al arrancar Linux.

Los resultados corresponden al dispositivo probado. El código declara un eje de presión de 0 a 8191, **pero eso no demuestra que se hayan medido 8192 niveles efectivos**.

### Requisitos

- Linux con acceso al dispositivo USB y a `/dev/uinput`.
- Una tableta compatible; comprobá primero el identificador USB.
- Rust/Cargo y dependencias necesarias para compilar `rusb` y `evdev`.

En Arch Linux:

```bash
sudo pacman -S --needed git base-devel rust cargo pkgconf libusb
```

Si tenés `lsusb` (paquete `usbutils`), comprobá:

```bash
lsusb -d 08f2:6811
```

### Instalación en Arch Linux

```bash
git clone https://github.com/miasa-goma/Tablet-VINSA-1060-Plus-Linux-Driver.git
cd Tablet-VINSA-1060-Plus-Linux-Driver/driver
cargo build --release
```

Para una prueba manual:

```bash
sudo ./target/release/v1060p
```

Dejá la terminal abierta mientras probás la presión, el mapeo y los trazos en Krita. Detené el proceso con `Ctrl+C` antes de iniciar un servicio automático. En esta configuración se utiliza `sudo` para acceder al USB y a `uinput`; **no hace falta aplicar reglas genéricas con permisos `0666`**.

### Inicio automático con systemd (opcional)

Desde el directorio `driver` que compilaste:

```bash
sudo install -Dm755 target/release/v1060p /usr/local/bin/gadnic-vinsa
sudo nano /etc/systemd/system/gadnic.service
```

Pegá lo siguiente en el archivo:

```ini
[Unit]
Description=Gadnic VINSA drawing tablet driver
After=systemd-udevd.service
StartLimitIntervalSec=0

[Service]
Type=simple
ExecStart=/usr/local/bin/gadnic-vinsa
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Guardá el archivo. **Cerrá cualquier instancia manual** del controlador antes de activar el servicio:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now gadnic.service
systemctl status gadnic.service
```

Para consultar errores:

```bash
journalctl -u gadnic.service -n 50 --no-pager
```

Para detenerlo o desactivar su inicio automático:

```bash
sudo systemctl stop gadnic.service
sudo systemctl disable gadnic.service
```

Si actualizás el código, volvé a compilar, detené el servicio, instalá el nuevo ejecutable y reinicialo con `sudo systemctl restart gadnic.service`.

### Controles y pruebas

El proyecto original ofrece **modo mouse** y **modo tableta** (botón **B**), además de ajuste del área del mouse con **[** y **]**. El fork inicia en **modo tableta**.

En Krita comprobá:

1. Que el cursor siga la posición del lápiz.
2. Que un trazo suave y uno fuerte respondan a la presión.
3. Que las líneas y curvas rápidas no tengan ondulaciones excesivas.
4. Que mover el lápiz por encima de la superficie **no** dibuje.

Si algo falla, confirmá que solamente haya una instancia del driver ejecutándose y consultá `journalctl`. No se garantiza compatibilidad universal.

### Archivos modificados

La adaptación de este fork está en [`driver/src/virtual_device.rs`](driver/src/virtual_device.rs). El resto del proyecto proviene del trabajo original.

---

## English

### What's improved in this fork?

On the tested device:

- **More stable fast strokes:** tablet mode sends pen coordinates without the former weighted smoothing, reducing lag-induced wobble. Mouse mode retains smoothing.
- **Ghost-stroke handling:** the pressure normalization treats `raw_pressure == 0` as no contact.
- **Pressure-sensitive drawing:** the pressure behavior was tested successfully in Krita.
- **Absolute tablet mapping:** retains the usable full-area tablet mode.
- **Optional automatic startup:** a `systemd` service example is provided below.

These are **observations from one tested device**, not a guarantee across models. The virtual pressure axis is declared as 0–8191; **8192 effective pressure levels have not been independently measured**.

### Requirements

- Linux with access to the USB device and `/dev/uinput`.
- A compatible drawing tablet. Check the USB ID first.
- Rust/Cargo and build dependencies for `rusb` and `evdev`.

On Arch Linux:

```bash
sudo pacman -S --needed git base-devel rust cargo pkgconf libusb
```

If `lsusb` is installed (from `usbutils`), check:

```bash
lsusb -d 08f2:6811
```

### Build and run

```bash
git clone https://github.com/miasa-goma/Tablet-VINSA-1060-Plus-Linux-Driver.git
cd Tablet-VINSA-1060-Plus-Linux-Driver/driver
cargo build --release
sudo ./target/release/v1060p
```

Keep the terminal open during testing, and press `Ctrl+C` to stop the manual driver before enabling the service. This setup runs with `sudo` for USB and `uinput` access; **world-writable (`0666`) device rules are not required**.

### Start automatically using systemd (optional)

From the compiled `driver` directory:

```bash
sudo install -Dm755 target/release/v1060p /usr/local/bin/gadnic-vinsa
sudo nano /etc/systemd/system/gadnic.service
```

Add:

```ini
[Unit]
Description=Gadnic VINSA drawing tablet driver
After=systemd-udevd.service
StartLimitIntervalSec=0

[Service]
Type=simple
ExecStart=/usr/local/bin/gadnic-vinsa
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Stop any manually running copy before starting the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now gadnic.service
systemctl status gadnic.service
```

Troubleshooting:

```bash
journalctl -u gadnic.service -n 50 --no-pager
```

To stop and disable the service:

```bash
sudo systemctl stop gadnic.service
sudo systemctl disable gadnic.service
```

After rebuilding, stop the service, reinstall the binary, and restart it with `sudo systemctl restart gadnic.service`.

### Tablet controls and validation

The upstream driver provides a **B** button toggle between mouse/tablet modes, and **[** / **]** to adjust the mouse-mode working area. This fork defaults to **tablet mode**.

In Krita, verify cursor mapping, light/heavy pressure, fast strokes, and that hovering does not paint. If problems occur, check for multiple driver instances and inspect the service logs.

### Modified code

The community changes are in [`driver/src/virtual_device.rs`](driver/src/virtual_device.rs). Other components are inherited from upstream.

---

## Credits / Créditos

- **Original VINSA 1060 Plus Linux driver:** [feveal](https://github.com/feveal/Tablet-VINSA-1060-Plus-Linux-Driver).
- **Earlier research:** [marvinbelfort / mx002_linux_driver](https://github.com/marvinbelfort/mx002_linux_driver).
- **Full-mode research:** [DIGImend/10moons-tools](https://github.com/DIGImend/10moons-tools).
- **User-space driver approach:** [alex-s-v/10moons-driver](https://github.com/alex-s-v/10moons-driver).
- **Community testing and modifications:** [miasa-goma](https://github.com/miasa-goma).

**License / Licencia:** [GNU GPL-3.0](LICENSE). This fork retains the upstream license and attribution.
