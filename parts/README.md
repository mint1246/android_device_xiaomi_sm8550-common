# XiaomiParts

Imported from [crDroid's SM8550 common tree, branch 17.0](https://github.com/crdroidandroid/android_device_xiaomi_sm8550-common/tree/17.0/parts), at commit `c37e036c1a2e22eee5ded57815609d743b897e59`.
The original source copyrights and licenses are retained.

The app installs in `system_ext` as `org.lineageos.settings`, is signed with the
platform certificate, and uses the system UID. Its activities integrate with
Settings, with quick settings tiles for supported controls.

## Integration

- `common.mk` includes `XiaomiParts`; its required modules install its permission
  allowlist and `init/init.xiaomiparts.rc`.
- The `xiaomiparts_app` SELinux domain is selected by the exact package name in
  `sepolicy/private/seapp_contexts`. It is exported through system_ext public
  policy so vendor policy can allow the existing hardware sysfs labels.
- `persist.sys.parts.*`, `sys.perf_mode_active`, and `sys.thermal.controlled_by`
  have a dedicated property type. CPU/GPU settings are applied by system init;
  the app has no direct CPU/GPU sysfs write permissions.
- Touch edge rejection and orientation have their own sysfs label. Thermal,
  brightness, and charge-current controls use the existing Qualcomm labels.
- The GPU MHz properties have separate init triggers, so setting a frequency
  does not race the subsequent MHz property update.
- The speaker-cleaning activity is exported so Settings can open it. The
  internal touch-orientation service is not exported.

The earlier local `batterysecret.te` change is not part of this import. The
charging controls here adjust the current limit after detecting HyperCharge;
they do not replace charger authentication or guarantee a particular wattage.

## Validation

Build for Vermeer with:

```sh
source build/envsetup.sh
lunch lineage_vermeer-cp2a-userdebug
m -j4 XiaomiParts selinux_policy sepolicy_tests
```

On a build installed on the phone, verify Settings entries and tiles, persistence
after reboot, thermal profile switching, brightness, edge rejection on rotation,
speaker cleaning, and charge limiting with a supported charger. Check the
`xiaomiparts_app` process context and SELinux denials while exercising each
control. CPU/GPU options and fixed frequency tables are inherited from crDroid
and still need verification against the running kernel.

Import validation (2026-10-17): `selinux_policy` and `sepolicy_tests` built
successfully for `lineage_vermeer-cp2a-userdebug`. The init script passed
`host_init_verifier`. Current resources, merged manifests, generated R classes,
and all 34 Java sources compiled successfully using cached framework/library
dependencies. A full app/ROM dependency rebuild and on-device checks remain
pending.
