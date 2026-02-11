# NetBSD Audio Setup with sndio - Step by Step Summary

## 1. Install sndio
```bash
pkgin install sndio
```

## 2. Identify Audio Devices
```bash
audiocfg list
```
Showed two devices:
- `audio0` - Intel HDMI/DP (not needed for regular audio)
- `audio1` - Realtek ALC671 (the one we want)

## 3. Set Realtek as Default Audio Device (as root)
```bash
audiocfg default 1
```
Verify with `audiocfg list` - `[*]` should now be on audio1

## 4. Make Default Persistent Across Reboots
```bash
echo "audio1=YES" >> /etc/rc.conf
```

## 5. Install the sndio RC Service Script
The service script is in pkgsrc examples, not in `/etc/rc.d/` by default:
```bash
cp /usr/pkg/share/examples/rc.d/sndio /etc/rc.d/
chmod +x /etc/rc.d/sndio
```

## 6. Enable and Start sndio Service
```bash
echo 'sndio=YES' >> /etc/rc.conf
/etc/rc.d/sndio start
```

## 7. Configure sndio to Use the Correct Device (optional)
Either system-wide:
```bash
echo 'sndiod_flags=-f rsnd/1' >> /etc/rc.conf
/etc/rc.d/sndio restart
```

Or per-user (`~/.sndio.conf`):
```
sndiod_flags=-f rsnd/1
```

## 8. Test Audio
Direct test (without sndio):
```bash
cat /dev/urandom > /dev/audio
```

Test sndio specifically:
```bash
aucat -i /dev/urandom -f rsnd/1
```

## 9. Adjust and Save Volume Levels
```bash
mixerctl -a                    # list all controls
mixerctl -w outputs.master=255,255  # set volume (0-255)
/etc/rc.d/mixerctl save        # save to /etc/mixerctl.conf
```

## Key NetBSD Takeaways:
- **`audiocfg`** - manage audio devices, set default
- **`mixerctl`** - adjust volume/mixer settings
- **pkgsrc services** live in `/usr/pkg/share/examples/rc.d/`, need to be copied to `/etc/rc.d/`
- **`/etc/rc.conf`** - enable services with `service=YES`
- **`/etc/rc.d/service start|stop|restart|status`** - control services
- **audio persistence** requires both device default (`audio1=YES`) and mixer settings (`mixerctl save`)

---

This same pattern applies to many other pkgsrc services (nginx, mysql, etc.) - 
copy the rc.d script, enable in rc.conf, start the service.
