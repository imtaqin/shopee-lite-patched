# Shopee Auto-Spoof — Magisk module

[`modules/shopee-autospoof-1.0.zip`](../modules/shopee-autospoof-1.0.zip)

Auto device-fingerprint spoof untuk paket Shopee khusus emulator — pengganti
step *manual spoof* di GUI (`shopee_bypass_qt.step_manual_spoof`), jalan
otomatis tiap boot, tanpa perlu perintah `resetprop` manual.

## Isi

- **post-fs-data.sh** — anti-emu kernel-level lebih awal:
  `ro.kernel.qemu=0`, `ro.kernel.qemu.gles=0`, `ro.hardware=qcom`,
  `ro.boot.hardware=qcom`, `ro.boot.selinux=enforcing`,
  bind-mount `fake_cpuinfo` ke `/proc/cpuinfo` (dibaca native libshpssdk).
- **service.sh** — fingerprint penuh setelah `boot_completed` (tulisan prop di
  stage ini bersih; di post-fs-data bisa korup untuk prop konteks-definisi).
- **fake_cpuinfo** — CPU Qualcomm SM7250 (dipakai kedua profil).

## Profil fingerprint

| Profil | Isi | Catatan |
|---|---|---|
| `pixel5` (default) | `google/redfin/redfin:14/UP1A.231005.007/10754064:user/release-keys` | **Fingerprint factory asli bersertifikat** (dump asli Google Pixel 5). Koheren 1:1 dengan `fake_cpuinfo` (Pixel 5 = Snapdragon 765G = SM7250), `first_api_level=30`, patch `2023-10-05`. |
| `tecno` | `TECNO/BD4-OP/TECNO-BD4:14/UP1A.231005.007/173010:user/release-keys` | Profil bawaan GUI (TECNO BD4, pasar Indonesia). |

Kedua profil menyetel `user/release-keys`, `security_patch=2023-10-05`, dan
`cpu.abilist=arm64-v8a,armeabi-v7a,armeabi` secara konsisten.

## Install

```sh
adb push modules/shopee-autospoof-1.0.zip /data/local/tmp/
adb shell su -c 'magisk --install-module /data/local/tmp/shopee-autospoof-1.0.zip'
adb reboot
```

## Ganti profil

```sh
adb shell su -c 'echo tecno > /data/adb/modules/shopee_autospoof/profile'
adb reboot
```

Tanpa file `profile` (atau isinya selain `tecno`) → pakai `pixel5`.

## Verifikasi

```sh
adb shell "getprop ro.build.fingerprint; head -2 /proc/cpuinfo; tail -1 /proc/cpuinfo"
# expected (pixel5):
#   google/redfin/redfin:14/UP1A.231005.007/10754064:user/release-keys
#   Processor : AArch64 Processor rev 12 (aarch64)
#   Hardware  : Qualcomm Technologies, Inc SM7250
```

## Catatan

- Nilai fingerprint diverifikasi dari dump factory asli
  (`gm-stuffs/google_redfin_dump`, branch
  `redfin-user-14-UP1A.231005.007-10754064-release-keys`).
- Modul menggantikan `shopee-emu-mask` (v1.0/v2) — tidak perlu keduanya.
- Install paket split emulator tetap seperti biasa:
  `adb install-multiple base-487-lspatched.apk split_config.arm64_v8a.apk split_config.xhdpi.apk`.
