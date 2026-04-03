Setup Local Multicast for ka9q-radio

To set up a local multicast environment for **ka9q-radio**, follow these key steps:

### **1. Install and Configure `avahi-daemon`**
- Install Avahi for local DNS name resolution (required for multicast names like `hf.local`):
  ```bash
  sudo apt install avahi-daemon
  sudo systemctl start avahi-daemon.service
  sudo systemctl status avahi-daemon.service
  ```
- This enables the use of `.local` hostnames for multicast streams, avoiding manual IP configuration.

### **2. Set Up `radiod` Configuration File**
- Use a config file like `radiod@rx888.conf` (or similar for your hardware).
- Ensure the `[global]` section includes:
  ```ini
  [global]
  hardware = rx888
  status = hf.local
  data = hf-data.local
  samprate = 12000
  mode = usb
  ttl = 0
  ```
  - `ttl = 0`: Keeps multicast traffic local (ideal for single-machine use).
  - `status` and `data` names are hashed to multicast IPs in `239.0.0.0/8` block.
  - `data = hf-data.local` defines the multicast stream name.

### **3. Optimize FFT Performance**
- Generate a **FFT "wisdom" file** to optimize performance:
  ```bash
  time fftwf-wisdom -v -T 1 -o wisdom rof500000 cof36480 cob1920 cob1200 cob960 cob800 cob600 cob480 cob320 cob300 cob200 cob160
  ```
- Move the resulting `wisdom` file to `/etc/fftw/` and back up the old one.

### **4. Use Local Loopback (Recommended for Single Machine)**
- With `ttl = 0`, all traffic stays on the local machine.
- Use `pcmrecord` or `opusd` to consume streams locally:
  ```bash
  pcmrecord -i hf-data.local -o - | opusenc - - | play -
  ```
- This avoids network issues and reduces latency.

### **5. Avoid WiFi for Multicast**
- **Critical**: WiFi networks often handle multicast poorly.
- Use a **wired Ethernet connection** (e.g., `eth0`) and avoid `wlan0`.
- If using a Raspberry Pi, ensure `eth0` is used as the default interface to prevent multicast issues.

### **6. Verify Setup**
- Run `radiod` with your config:
  ```bash
  sudo radiod -c /etc/radio/radiod@rx888.conf
  ```
- Check that multicast streams appear via:
  ```bash
  sudo netstat -g
  ```
- Use `tcpdump` to verify traffic on multicast addresses:
  ```bash
  sudo tcpdump -i eth0 -n -s 0 -c 10 'udp port 5004'
  ```

> ✅ **Best Practice**: Use `ttl = 0` and `avahi` for a reliable, low-latency local setup. Avoid multicast over WiFi or unmanaged switches.
