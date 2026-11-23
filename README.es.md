🌐 *Leer en [Inglés](README.md)*

# Migración de Teléfonos Cisco (Serie 79xx) de SCCP a SIP y Pack de Firmwares

Archivos de firmware, plantillas de configuración y guía paso a paso para convertir teléfonos IP **Cisco 7906, 7911, 7940, 7941, 7942, 7945, 7960, 7961, 7962 y 7965** de **SCCP (Skinny)** a **SIP** para utilizarlos con **Issabel PBX / Asterisk**.

## 📂 Estructura del Repositorio

* **`common-config/`**: Archivos base de configuración (`SIPDefault.cnf`, `dialplan.xml`, `ringlist.xml`).
* **`cisco-7906-7911/`**: Archivos de firmware (serie `06` / `11`) organizados por pasos de actualización:
  * `step1-sccp-8.4`: Firmware puente obligatorio en SCCP para versiones antiguas (`< 8.3`).
  * `step1.1-sccp9.21`: Firmware puente opcional en SCCP (`9.21`) si falla el salto directo de `8.4` a SIP.
  * `step2-sip-9.x`: Firmware final en modo SIP.
* **`cisco-7940-7960/`**: Firmwares `P003` (SCCP) y `P0S3` (SIP `8-12-00`).
* **`cisco-7941-7961/`**: Archivos de firmware para modelos `7941/7961` (serie `41` / `61`, SIP `9-2-1S`).
* **`cisco-7942-7962/`**: Archivos de firmware para modelos `7942/7962` (serie `42` / `62`).
* **`cisco-7945-7965/`**: Archivos de firmware para modelos `7945/7965` (serie `45` / `65`).

---

## ⚠️ Advertencia Crítica: Ruta de Actualización y Diagnóstico

**NO** intentes cargar un firmware SIP `9.x` directamente en teléfonos que tengan versiones SCCP muy antiguas (como `7.x` u `8.0`), especialmente en modelos como el **Cisco 7906 / 7911** o **7941 / 7942**. Los *bootloaders* antiguos no reconocen las firmas digitales nuevas.

**Rutas de actualización probadas (`cisco-7906-7911/`):**
* **Ruta estándar (2 pasos):** `SCCP Antiguo (< 8.3)` ➔ **`step1-sccp-8.4`** ➔ **`step2-sip-9.x`**
* **Ruta extendida (3 pasos, si falla el salto directo desde 8.4 a SIP):** `SCCP Antiguo (< 8.3)` ➔ **`step1-sccp-8.4`** ➔ **`step1.1-sccp9.21`** ➔ **`step2-sip-9.x`**

> **💡 ¿Cómo saber si la actualización falló?** Observa la pantalla del teléfono durante el proceso: si la barra de progreso salta **de inmediato de 0% a 100%** sin mostrar una carga gradual, significa que el teléfono rechazó el firmware por incompatibilidad de versión/firma. Aplica primero el paso intermedio en SCCP (`step1-sccp-8.4` o `step1.1-sccp9.21`) antes de volver a intentar el paso a SIP.

---

## 🚀 Guía de Migración Paso a Paso (TFTP + Issabel PBX)

### 1. Preparar el Servidor TFTP
1. Instala un servidor TFTP en tu PC (por ejemplo, **Tftpd64** en Windows o **tftpd-hpa** en Linux) o utiliza el directorio `/tftpboot` de Issabel.
2. Copia los archivos de firmware de la carpeta de tu modelo (por ejemplo, primero `cisco-7906-7911/step1-sccp-8.4/` y luego `step2-sip-9.x/`) junto con los archivos de `common-config/` en la raíz de tu carpeta TFTP.
3. Configura tu servidor DHCP para entregar la **Opción 150** (o **Opción 66**) apuntando a la IP de tu servidor TFTP.

### 2. Reset de Fábrica del Teléfono Cisco
Para borrar configuraciones previas y forzar al teléfono a solicitar los nuevos archivos por TFTP:
1. Desconecta el cable de corriente (o PoE) del teléfono.
2. Mantén presionada la tecla **`#`** mientras vuelves a conectar la alimentación.
3. Sigue presionando **`#`** hasta que los botones laterales o LEDs de línea parpadeen en secuencia.
4. Suelta **`#`** y marca rápidamente **`123456789*0#`** en el teclado.
5. El teléfono borrará su memoria y descargará los archivos desde el TFTP.

### 3. Aprovisionamiento con Issabel Endpoint Configurator
Una vez que el teléfono tenga el firmware SIP cargado (o esté listo para descargar su `SEP<MAC>.cnf.xml`):
1. Entra a la interfaz web de **Issabel PBX** y ve a **PBX > Endpoint Configurator**.
2. Escanea la red o agrega el teléfono manualmente indicando su **Dirección MAC** y seleccionando el modelo Cisco correspondiente.
3. Asigna la extensión SIP y guarda los cambios. Issabel generará automáticamente el archivo `SEP<MAC>.cnf.xml` dentro de `/tftpboot`.
4. Apunta la **Opción 150 / 66** de tu DHCP hacia la IP de tu servidor Issabel y reinicia el teléfono.

### 4. Ajuste Permanente de Fecha y Zona Horaria en Issabel
Por defecto, el *Endpoint Configurator* de Issabel genera un formato de fecha y zona horaria que el firmware SIP de Cisco puede mostrar desfasado. Puedes modificar directamente la plantilla maestra de Issabel (`/usr/share/issabel/endpoint-classes/tpl/Cisco_local_SEP.tpl`) por SSH para que todos los teléfonos generados hereden la configuración correcta:

```bash
# Crear respaldo y ajustar formato de fecha a Día/Mes/Año (12h AM/PM)
sed -i.bak 's|<dateTemplate>[^<]*</dateTemplate>|<dateTemplate>D/M/YA</dateTemplate>|g' /usr/share/issabel/endpoint-classes/tpl/Cisco_local_SEP.tpl

# Configurar zona horaria válida para Cisco (ej. SA Western Standard Time para GMT-4 / Caracas)
sed -i 's|<timeZone>[^<]*</timeZone>|<timeZone>SA Western Standard Time</timeZone>|g' /usr/share/issabel/endpoint-classes/tpl/Cisco_local_SEP.tpl
```

Luego de ejecutar los comandos, vuelve a **PBX > Endpoint Configurator** en Issabel, regenera la configuración de tus teléfonos y reinícialos.

---

## 📜 Aviso Legal
Los archivos de firmware de Cisco son propiedad de Cisco Systems, Inc. Se comparten únicamente con fines educativos, de laboratorio y recuperación de hardware legado.
