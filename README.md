### Structure

```
devices/
    {CHIP NAME}/
        overlays/
            {DEVICE NAME} <-- device specific data
        base/
```

### Data structure

At boot time, `/tmp/qcom-hexagon-fs` is created from all of the below:
  - the chip's base folder (eg: `$SENSOR_FIRMWARE_REPO/devices/SM8750/base`)
  - the device's overlay folder (eg: `$SENSOR_FIRMWARE_REPO/devices/SM8750/overlays/AYN Odin 3`)
  - a config folder on your device (`/storage/.config/qcom-hexagon-fs`)

### Sourcing

The sensor data can be found in your the Android partitions. (probably)

For the AYN Odin 3 (SM8750), this is at: `vendor/etc/sensors/` and `persist/sensors/`

To find out where to put these, boot your device into ROCKNIX and see what adsprpcd is complaining about:

```
# journalctl | grep fastrpc
apps_std_imp.c:1720:Error 0x201: apps_std_stat: failed to stat /sns_odp_config_params.bin, file open returned 0x2 (No such file or directory)
mod_table.c:909: Error 0x2: open_mod_table_handle_invoke failed for handle:0x8b4e69b0, sc:0x1f020100
apps_std_imp.c:1125: Warning: apps_std_fopen_with_env failed with 0x2 for /mnt/vendor/persist/sensors/qmi8658_usid.txt (No such file or directory)
mod_table.c:909: Error 0x2: open_mod_table_handle_invoke failed for handle:0x8b4e69b0, sc:0x13050100
apps_std_imp.c:1125: Warning: apps_std_fopen_with_env failed with 0x2 for /mnt/vendor/persist/sensors/qmi8658_coid.txt (No such file or directory)
mod_table.c:909: Error 0x2: open_mod_table_handle_invoke failed for handle:0x8b4e69b0, sc:0x13050100
apps_std_imp.c:1125: Warning: apps_std_fopen_with_env failed with 0x2 for /mnt/vendor/persist/sensors/qmi8658_coid.txt (No such file or directory)
mod_table.c:909: Error 0x2: open_mod_table_handle_invoke failed for handle:0x8b4e69b0, sc:0x13050100
apps_std_imp.c:1125: Warning: apps_std_fopen_with_env failed with 0x2 for /mnt/vendor/persist/sensors/acc_gyro_regdata.txt (No such file or directory)
mod_table.c:909: Error 0x2: open_mod_table_handle_invoke failed for handle:0x8b4e69b0, sc:0x13050100
```

### Adding the data

**NOTE**: If the data is specific to the DEVICE (eg. AYN Odin 3) and not the CHIP (eg. SM8750) put it in the device's overlay folder

**NOTE**: Ignore the `/mnt/` at the start of paths, we bind `/tmp/qcom-hexagon-fs/vendor` to `/tmp/qcom-hexagon-fs/mnt/vendor` at runtime.

As an example, for the Odin 3, the process looks like:

- We see **adsprpcd** is looking for a file at `/mnt/vendor/persist/sensors/acc_gyro_regdata.txt` (virtual path)
- The virtual path `/mnt/vendor` corresponds to the repo paths:
  - `devices/SM8750/overlays/AYN Odin 3/vendor/` (for device specific data)
  - `devices/SM8750/base/vendor` (for chip generic data)
- We find `acc_gyro_regdata.txt` in Android's **persist** partition: `/persist/sensors/acc_gyro_regdata.txt`
- `acc_gyro_regdata.txt` seems like runtime data for this specific device
- Copy it to `devices/SM8750/overlays/AYN Odin 3/vendor/persist/sensors/acc_gyro_regdata.txt`
