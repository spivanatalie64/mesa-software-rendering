# mesa-software-rendering

Silence `MESA-EGL: warning: DRI3 error: Could not get DRI3 device` on machines
whose only GPU predates usable 3D acceleration, by telling Mesa up front to
render in software.

## The symptom

```text
$ pavucontrol
MESA-EGL: warning: DRI3 error: Could not get DRI3 device
MESA-EGL: warning: Ensure your X server supports DRI3 to get accelerated rendering
```

## The cause chain

1. Every EGL client probes the X server for the **DRI3** extension to get a
   direct-rendering device for hardware acceleration.
2. The GPU here is an **ATI/AMD ES1000** — a ~2007 server-management display
   chip (HP ProLiant BMC-adjacent hardware) with no usable 3D engine. Its
   kernel driver (`radeon`) cannot provide a modern render node.
3. EGL's DRI3 probe fails and Mesa logs the warning before falling back.
   It is cosmetic, but it appears on every GL/EGL app start.

No driver update can fix this: there is no 3D hardware to drive.

## The fix

Tell Mesa to skip the hardware path entirely:

```console
# echo 'LIBGL_ALWAYS_SOFTWARE=1' >> /etc/environment
```

`/etc/environment` is read by pam_env at login, so it applies to every future
session without touching shell rc files. Re-login, then verify:

```console
$ glxinfo -B        # pacman -S mesa-utils
...
Device: llvmpipe (LLVM x.y.z, 256 bits)
```

`llvmpipe` is Mesa's multithreaded CPU rasterizer. For desktop apps
(pavucontrol, GTK/Qt tooling, terminals) it is indistinguishable from the real
thing on any CPU of the last 15 years.

## When NOT to do this

* On machines with a working GPU this forces slow software rendering — set it
  per-app instead: `LIBGL_ALWAYS_SOFTWARE=1 someapp`.
* If you later add a real graphics card, remove the line.

MIT licensed; see [LICENSE](LICENSE).
