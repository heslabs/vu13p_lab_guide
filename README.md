# vu13p lab guide

``
ValueError: bad marshal data (invalid reference)
```

Resolve the issue in the following order.

1. **Clear the corrupted bytecode cache for the system `sympy` package.**

   ```bash
   $ sudo find /usr/lib/python3/dist-packages/sympy -name "__pycache__" -exec rm -rf {} +
   ```

   Re-run the script. Python recompiles fresh `.pyc` files automatically, and the import error is typically resolved.

2. **If the import still fails, install `sympy` directly inside the venv** so it no longer falls back to the system copy.

   ```bash
   $ source /usr/local/share/pynq-venv/bin/activate
   $ pip install sympy --force-reinstall
   ```

3. **Confirm whether the venv was created with access to system-wide packages.**

   ```bash
   $ grep include-system-site-packages /usr/local/share/pynq-venv/pyvenv.cfg
   ```

   If this is set to `true` and system packages are not required, set it to `false` and reinstall the needed packages inside the venv to prevent the venv from reaching outside itself.

> **Note:** On PYNQ boards, corrupted `.pyc` files are commonly caused by an unclean shutdown or a filesystem that ran out of space mid-write. If this issue recurs, also check available disk space:
>
> ```bash
> $ df -h
