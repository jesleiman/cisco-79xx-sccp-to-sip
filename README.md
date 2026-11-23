🌐 *Read this in [Spanish](README.es.md)*

# Cisco IP Phone (79xx Series) SCCP to SIP Migration & Firmware Pack

Firmware files, configuration templates, and step-by-step migration guide to convert **Cisco 7906, 7911, 7940, 7941, 7942, 7945, 7960, 7961, 7962, and 7965** IP Phones from **SCCP (Skinny)** to **SIP** for use with **Issabel PBX / Asterisk**.

## 📂 Repository Structure

* **`common-config/`**: Base configuration files (`SIPDefault.cnf`, `dialplan.xml`, `ringlist.xml`).
* **`cisco-7906-7911/`**: Firmware loads (`06` / `11` series) organized in upgrade steps:
  * `step1-sccp-8.4`: Mandatory intermediate SCCP load for legacy firmware (`< 8.3`).
  * `step1.1-sccp9.21`: Optional secondary SCCP bridge load (`9.21`) if direct jump from `8.4` to SIP fails.
  * `step2-sip-9.x`: Final SIP firmware load.
* **`cisco-7940-7960/`**: Legacy `P003` (SCCP) and `P0S3` (SIP `8-12-00`) firmware files.
* **`cisco-7941-7961/`**: Firmware loads for `7941/7961` (`41` / `61` series, SIP `9-2-1S`).
* **`cisco-7942-7962/`**: Firmware loads for `7942/7962` (`42` / `62` series).
* **`cisco-7945-7965/`**: Firmware loads for `7945/7965` (`45` / `65` series).

---

## ⚠️ Critical Warning: Firmware Upgrade Path & Troubleshooting

Do **NOT** attempt to flash SIP `9.x` firmware directly onto phones running very old SCCP versions (such as `7.x` or `8.0`), especially on models like the **Cisco 7906 / 7911** or **7941 / 7942**. Older bootloaders cannot verify newer digital signatures.

**Tested step-by-step upgrade paths (`cisco-7906-7911/`):**
* **Standard 2-Step Path:** `Old SCCP (< 8.3)` ➔ **`step1-sccp-8.4`** ➔ **`step2-sip-9.x`**
* **Extended 3-Step Path (if direct jump from 8.4 to SIP fails):** `Old SCCP (< 8.3)` ➔ **`step1-sccp-8.4`** ➔ **`step1.1-sccp9.21`** ➔ **`step2-sip-9.x`**

> **💡 How to tell if the upgrade failed:** Watch the progress bar on the phone screen. If the upgrade jumps **instantly from 0% to 100%** without showing a gradual download progression, the phone rejected the firmware signature. Step down to the intermediate SCCP version (`step1-sccp-8.4` or `step1.1-sccp9.21`) before trying SIP again.

---

## 🚀 Step-by-Step Migration Guide (Using TFTP + Issabel PBX)

### 1. Prepare the TFTP Server
1. Install a TFTP server on your PC (e.g., **Tftpd64** on Windows or **tftpd-hpa** on Linux) or use Issabel's built-in `/tftpboot` directory.
2. Copy the firmware files from your phone's folder (e.g., `cisco-7906-7911/step1-sccp-8.4/` first, then `step2-sip-9.x/`) along with the files inside `common-config/` into the root of your TFTP folder.
3. Configure your DHCP server to deliver **Option 150** (or **Option 66**) pointing to the IP address of your TFTP server.

### 2. Factory Reset the Cisco Phone
To force the phone to clear old settings and request new files from TFTP:
1. Unplug the power (or PoE) cable.
2. Press and hold the **`#`** key while plugging the power back in.
3. Keep holding **`#`** until the headset/mute/speaker buttons or line LEDs flash in sequence.
4. Release **`#`** and quickly type **`123456789*0#`** on the keypad.
5. The phone will wipe its configuration and begin downloading the loads from your TFTP server.

### 3. Provisioning with Issabel Endpoint Configurator
Once the phone is running SIP firmware (or ready to fetch its `SEP<MAC>.cnf.xml`):
1. Open **Issabel PBX** web interface and go to **PBX > Endpoint Configurator**.
2. Scan your network or manually add the phone using its **MAC Address** and select the corresponding Cisco model.
3. Assign the SIP extension and click **Save / Generate**. Issabel will automatically create the `SEP<MAC>.cnf.xml` file in `/tftpboot`.
4. Point DHCP **Option 150 / 66** to your Issabel server IP and reboot the phone.

### 4. Fixing Date Format & Time Zone Permanently in Issabel
By default, Issabel's Endpoint Configurator generates a date/time template that Cisco SIP firmware may display incorrectly. You can patch Issabel's master template (`/usr/share/issabel/endpoint-classes/tpl/Cisco_local_SEP.tpl`) directly via SSH so every newly provisioned phone inherits the right settings:

```bash
# Backup original template and set Date Format to Day/Month/Year (12h AM/PM)
sed -i.bak 's|<dateTemplate>[^<]*</dateTemplate>|<dateTemplate>D/M/YA</dateTemplate>|g' /usr/share/issabel/endpoint-classes/tpl/Cisco_local_SEP.tpl

# Set Cisco Time Zone (e.g., SA Western Standard Time for GMT-4 / Caracas)
sed -i 's|<timeZone>[^<]*</timeZone>|<timeZone>SA Western Standard Time</timeZone>|g' /usr/share/issabel/endpoint-classes/tpl/Cisco_local_SEP.tpl
```

After running these commands, go back to **PBX > Endpoint Configurator** in Issabel, re-save/regenerate your endpoints, and reboot the phones.

---

## 📜 Disclaimer
Cisco firmware files are property of Cisco Systems, Inc. Provided here for educational, lab, and legacy hardware recovery purposes.


