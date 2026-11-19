# Corne v4.1: estado y contexto (2026-11-18)

## Teclado
- Corne v4.1 (foostan, PCB rev 4.1, chip RP2040), dividido en 2 mitades unidas por cable TRRS.
- Firmware de fábrica: **Vial oficial de foostan** (confirmado por el UID de Vial `{0x89,0x36,0x2A,0xC7,0xFA,0xD8,0x89,0x45}`).
- Se configura en https://vial.rocks. El usuario no quiere la app de escritorio de Vial, que no está firmada.
- **Cada mitad guarda su propio keymap.** El teclado usa el de la mitad conectada por USB, así que cada cambio de `.vil` hay que cargarlo en las dos mitades.

## Estado actual
- **Firmware v2 flasheado en las dos mitades** (2026-11-18, ~19:05; `picotool -v` OK). IDs de flash: derecha `0xC4607015C7670E23`, izquierda `0xC469ACA11F576835`.
- `corne-aerospace.vil` cargado en las dos mitades; USB en la izquierda.
- **Prueba de suspensión superada** (19:14–19:16, 4 ciclos de dormir y despertar, sin bucle).
- **Prueba de inactividad superada** (19:48–22:24, 2 h 36 min con la pantalla apagada, sin reposo del sistema porque `caffeinate`/`coreaudiod` lo impedían). Las dos mitades seguían funcionando. Con el firmware de fábrica, este mismo caso (dejarlo un rato sin tocar) hacía caer la mitad sin USB.
- Pendiente:
  - Probar una suspensión de 1 h o más y el uso diario.
  - El reinicio por watchdog todavía no se ha probado con una caída real de la conexión.

## Repo
- Fork `Roter-S/vial-qmk`, rama `corne` (rama por defecto del fork), clonado en `~/Web/vial-qmk`. El remote `upstream` apunta a `vial-kb/vial-qmk`.
- Clon superficial (`--depth 1`). Para traer actualizaciones de Vial, primero `git fetch --unshallow upstream`.
- Historial de la rama: fuentes de foostan tal cual → keymap v1 (heartbeat) → arreglo v2. Con `git log -p keyboards/foostan` se ve cada cambio.

## Archivos
- `keyboards/foostan/crkbd/`: fuentes de foostan, copiadas tal cual de `foostan/kbd_firmware@7e94242f` (`keyboards/crkbd/qmk/qmk_firmware`). Están separadas de `keyboards/crkbd`, que sigue a QMK upstream.
- `keyboards/foostan/crkbd/keymaps/vial_hb/`: keymap con heartbeat (`config.h`, `keymap.c`, `rules.mk`, `vial.json`). El v1 está en el commit `98d3a400` y el v2 en `7715fd6f`.
- `corne/layouts/corne-aerospace.vil`: layout actual. Cargarlo con *File → Load saved layout*.
- `corne/layouts/corne-original-backup.vil`: layout original de fábrica.
- `~/.config/corne` es un enlace simbólico a esta carpeta `corne/`. No hay otra copia.
- `corne/firmware/` (**solo local, ignorado por git** con `*.uf2`):
  - `corne-vial-heartbeat-v2.uf2`: **v2, el que está flasheado.**
  - `corne-vial-heartbeat.uf2`: v1, con el bug del bucle al dormir la Mac. No usar.
  - `corne-vial-official-foostan-2026-11-14.uf2`: firmware oficial más reciente, sin cambios (alternativa).
  - `backup-izq-2026-11-18.uf2` / `backup-der-2026-11-18.uf2`: volcado completo (2 MB) de cada mitad antes de flashear. **No se pueden regenerar: hay que guardarlos.** Para volver a fábrica: `picotool load <backup>.uf2` en esa mitad.

## Compilar
- Requisitos:
  - `brew`: `osx-cross/arm/arm-gcc-bin@8` (en `/opt/homebrew/opt/arm-gcc-bin@8/bin`) y `picotool`.
  - CLI `qmk` en un venv: `python3 -m venv <venv> && <venv>/bin/pip install qmk`.
- Submódulos: `git submodule update --init --recursive --depth 1`.
- Compilar:
  ```
  PATH=/opt/homebrew/opt/arm-gcc-bin@8/bin:<venv>/bin:$PATH make foostan/crkbd/rev4_1/standard:vial_hb
  ```
  Genera `foostan_crkbd_rev4_1_standard_vial_hb.uf2` en la raíz, ignorado por git.
- Vial genera un `BUILD_ID` aleatorio en cada compilación (`util/build_id.py`) y lo usa como "magic" de la EEPROM. Por eso dos builds nunca son idénticos y cada firmware nuevo borra el keymap guardado.
  - Para reproducir byte a byte el v1 o el v2 flasheados, `util/build_id.py` tiene que imprimir `#define BUILD_ID ((uint32_t)0x4BF1F1)`. No commitear ese cambio.
  - Verificado: con ese ID, el v2 compilado desde esta rama es idéntico a `corne-vial-heartbeat-v2.uf2`, y el v1 a `corne-vial-heartbeat.uf2`.
- Al comparar v1 y v2, de 197 objetos solo cambia `quantum/keymap_introspection.o` (ahí se compila `keymap.c`).

## Cómo flashear (el teclado NO tiene botones BOOT/RESET)
- Desconectar el USB y después el TRRS. **Nunca conectar ni desconectar el TRRS con el USB enchufado.** Dejar el TRRS conectado todo el tiempo también funciona: la otra mitad recibe corriente y se reinicia en bucle mientras tanto, lo cual es inofensivo.
- Entrar al bootloader con bootmagic: mantener **Q** (mitad izquierda) o **P** (mitad derecha) mientras se conecta el USB a esa mitad sola. Aparece el disco `RPI-RP2`. Es por posición física (matriz `[0,1]` / `[4,1]`), no por el keymap. **Borra el keymap de esa mitad.**
- `picotool info -a` (comprobar el ID de flash de la mitad), luego `picotool save --all <backup>.uf2`, luego `picotool load -v -x <firmware>.uf2`.
- Después: conectar el TRRS y luego el USB, y cargar el `.vil` en las dos mitades, cambiando el USB de mitad (el TRRS puede quedar conectado si el USB está desenchufado). Si Vial pide desbloquear: Tab + Q.

## Layout actual (resumen)
- Capa 0: `Caps` = `MO(4)` (capa AeroSpace). Pulgar izquierdo exterior = Cmd (antes Ctrl). Ctrl queda en la columna interior derecha.
- Capa 1 (`Fn1`, pulgar izquierdo): números, flechas en `HJKL`, `N`/`M`/`,` = mute/vol−/vol+, `C`/`V`/`B` = pista anterior/siguiente/play.
- Capa 2 (`Fn2`, pulgar derecho): símbolos. Capa 3 (`Fn1+Fn2`): RGB. Se quitó `QK_BOOT` porque el usuario lo activaba sin querer.
- Capa 4 (mantener `Caps`), atajos de AeroSpace:
  - `Q..P` = `alt-1..0` (ir a workspace).
  - `Z../` = `alt-shift-1..0` (mover ventana a workspace).
  - `HJKL` = foco; `ASDF` = mover ventana.
  - `Tab` = workspace anterior; `;` / `'` = resize.
  - `G` = captura al portapapeles (`Cmd+Ctrl+Shift+4`); tecla interior superior = captura a archivo (`Cmd+Shift+4`).
- `aerospace.toml` no se tocó: el teclado manda los mismos atajos que ya existían.

## Historia del problema: una mitad deja de funcionar
- Síntoma original (firmware de fábrica): después de un rato sin tocar el teclado, **deja de funcionar la mitad que no está conectada al USB**. Pasó con las dos mitades. A veces se cae todo el teclado. Se arregla desconectando y reconectando el USB.
- Descartado:
  - TRRS flojo (moverlo no corta la conexión).
  - RGB (falló con los LEDs apagados).
  - Cable USB malo y hub (el cable original pasa datos mal, pero falló igual con un cable bueno conectado directo a la Mac).
  - Suspensión de la Mac (falló con cargador, sin suspensión registrada en `pmset -g log`).
- Hipótesis: el firmware pierde la comunicación entre las dos mitades después de un rato inactivo. La mitad caída tenía los LEDs encendidos.
- Firmware de fábrica: build oficial más antiguo (68 KB, distinto del 2026-11-14), idéntico en las dos mitades. Su definición de Vial es idéntica a la de `vial.json`.

### v1: heartbeat (flasheado el 2026-11-18 por la mañana)
- La mitad con USB le manda una señal a la otra cada 500 ms. Si la otra mitad pasa 5 s sin recibirla, se reinicia sola (`mcu_reset`). `SPLIT_WATCHDOG` de QMK no sirve aquí, porque solo cubre la conexión inicial al encender.
- Verificado:
  - Compila sin warnings (`-Werror`).
  - El boot2 es idéntico al del firmware oficial.
  - UID, protocolo Vial 6 e identidad USB son iguales a los del firmware de fábrica.
  - `vial.json` validado con el parser KLE de Vial: 46/46 teclas coinciden con el `.vil`.
  - Se mantuvieron los 4 encoders: `restore_layout` de Vial no valida límites de encoders y fallaría si faltaran.

### Incidente con el v1 (2026-11-18, 18:30 y 18:51): bucle de reinicio al despertar la Mac
- Al despertar la Mac después de dormir (bastan 2 min), las dos mitades congelan sus LEDs y arrancan de nuevo cada ~3 s. Algunas teclas fallan. Reconectar el USB lo detiene.
- Verificado: la mitad con USB **no** se reinicia; la Mac la ve con el mismo ID todo el tiempo. La que se reinicia es la otra.
- Causa:
  - Durante la suspensión USB, QMK se queda en el bucle de suspensión de `protocol_pre_task` (`tmk_core/protocol/chibios/chibios.c`). `housekeeping_task_user` no corre y no sale ninguna señal, así que la mitad sin USB se reinicia a los 5 s.
  - En el RP2040, `mcu_reset()` = `NVIC_SystemReset()` (débil, en `platforms/chibios/bootloaders/rp2040.c`). Hipótesis no confirmada: solo reinicia los núcleos y no los periféricos (PIO/DMA). Verificado: ChibiOS (`hal_lld_init`) tampoco los reinicia al arrancar.
  - La mitad vuelve sin comunicación y el `SPLIT_WATCHDOG` de fábrica (`SPLIT_WATCHDOG_TIMEOUT` = `SPLIT_USB_TIMEOUT` + 100 = 2,1 s) la reinicia en bucle, siempre con el mismo tipo de reinicio.
  - La prueba del v2 confirma que ya no hay bucle al dormir, pero no aísla cuál de los dos cambios lo evitó: con la señal durante la suspensión, la mitad ya no se reinicia al dormir.

### v2: arreglo (flasheado el 2026-11-18, ~19:05)
- `mcu_reset()` redefinido con un reinicio por watchdog: `psm_hw->wdsel` = todo menos los osciladores (0x1fffc), luego `WATCHDOG_CTRL_TRIGGER`, igual que `watchdog_reboot()` de pico-sdk. Lo usan el heartbeat, `SPLIT_WATCHDOG` y `QK_REBOOT`.
- La señal se manda también desde `suspend_power_down_user`, que se llama en cada vuelta del bucle de suspensión (~17 ms). Ese bucle sigue escaneando la matriz y, con ella, el enlace entre mitades.
- Si alguna vez vuelve el bucle con el v2, el problema estaría en la comunicación de la mitad con USB. El siguiente paso sería que esa mitad se reinicie sola si pierde la conexión por varios segundos después de haberla tenido.
- Los LEDs que se encienden y el "pop" en los audífonos con la Mac dormida son despertares breves (DarkWake por Wi-Fi/Bluetooth): es comportamiento normal de macOS. Se pueden reducir desactivando "Activar para acceso a la red".

### Incidente con el v2 (2026-11-19, ~09:00–09:15): sin suspensión
- La Mac arrancó a las 08:56. Pocos minutos después la mitad derecha (sin USB) se quedó **sin LEDs** y dejó de funcionar. Reconectar el USB (izquierda) la recuperó.
- Un rato después fallaron las dos: lag, letras o espacios repetidos y borrado de más. Se recuperó solo, sin desconectar nada.
- Teclado conectado directo a la Mac, sin hub. No hubo reposo del sistema.
- Hipótesis (sin confirmar): contacto intermitente del TRRS (cable o jacks). La mitad sin USB recibe la corriente por el TRRS, y quedarse totalmente apagada no cuadra con un reinicio por el heartbeat. Las teclas repetidas encajan con transacciones split que fallan: el maestro tarda en mandar el "soltar" y macOS repite la tecla.
- Siguiente paso: un cable TRRS nuevo (4 polos). Si no, el contador de caídas para tener datos.

### Log de macOS del 2026-11-19 (clave)
- Se lee con `/usr/bin/log show ... --predicate 'process == "kernel" AND eventMessage CONTAINS "drd1"'`. Ojo: en zsh `log` es un builtin, hay que usar `/usr/bin/log`.
- La **mitad con USB (izquierda) deja de responder en el bus USB**: `transaction error` en sus endpoints, macOS la resetea (`terminateDevice: reset API call`) y la vuelve a enumerar, a veces mal (`enumeration failed`; queda como `IOUSBHostDevice` sin nombre ni HID).
- Pasó a las 09:03:57, 09:09:46–09:10:32 (muchos ciclos: el lag y las teclas repetidas) y 10:00:12 (quedó colgada sin configurar).
- No es el código del v2: `mcu_reset` solo lo llaman el heartbeat y `SPLIT_WATCHDOG`, y los dos solo actúan en la mitad sin USB.
- Hipótesis: caída de voltaje o reinicio de la mitad con USB (cable USB, puerto o un corto momentáneo en el TRRS). Que la derecha quede sin LEDs encaja con eso.
- **La izquierda falla en los dos papeles**: con USB en la izquierda se cae del bus; en una prueba anterior con USB en la derecha, la que dejó de funcionar a los minutos fue la izquierda. Con el firmware de fábrica también pasaba. Sospecha principal: **defecto de hardware en la mitad izquierda** (MCU, regulador, cristal o soldaduras).
- 12:30:50, 12:31:00 y 12:32:47: **mismo patrón en otro puerto de la Mac** (`drd2-port-hs@01100000`, otro controlador, desde las 10:07). El puerto de la Mac queda descartado.
- Desde que el USB va en la **derecha** (2026-11-19, tarde) no se han notado fallas. Encaja con la izquierda defectuosa. No está confirmado si el heartbeat la ha reiniciado: desde la Mac no se ve.
- **Garantía en trámite** (2026-11-19). Antes de enviar el teclado: restaurar el firmware de fábrica en cada mitad con su `backup-*.uf2`.
- Mitigación: USB en la **derecha**. Si la izquierda se cuelga como esclava, el heartbeat del v2 la reinicia sola y la derecha sigue funcionando.

## Mejoras posibles
1. **Contador de caídas.** La mitad con USB cuenta cada vez que pierde la conexión con la otra, y una tecla en la capa 3 escribe `caidas: N`. Sirve para saber si el heartbeat actuó o si simplemente no hubo falla.
2. **Watchdog de hardware del RP2040**, solo si el contador muestra caídas. Cubre un chip colgado y los casos en que se cae todo el teclado. Ojo: QMK se queda en el bucle de suspensión mientras la Mac duerme, así que hay que alimentar el watchdog también ahí para que no reinicie el teclado.
