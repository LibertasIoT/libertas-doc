# Try Libertas

Try a client first, with the **Libertas Cloud Hub**: a Hub hosted for you,
with nothing to set up. The current release is **Beta 2**.

<ul class="client-strip">
<li><a href="https://play.google.com/store/apps/details?id=com.smartonlabs.qwha"><img src="../assets/icons/android.svg" alt=""><strong>Android</strong><span>Libertas One on Google Play</span></a></li>
<li><div><img src="../assets/icons/apple.svg" alt=""><strong>iOS</strong><span>Coming to the App Store</span></div></li>
<li><a href="https://hub.smartonlabs.com"><img src="../assets/icons/web.svg" alt=""><strong>Web</strong><span>The Cloud Hub in your browser</span></a></li>
</ul>

In the App, tap **Add a new Hub** → **Libertas Cloud Hub**, then sign in in
the browser that opens. On the web, just sign in.

## Run your own Hub

Your own Hub runs Ubuntu 26.04, x86-64 or ARM64.

### Home Hub in Incus (Linux or Mac)

It runs on a computer at home, so phones on your Wi-Fi find it. You need Incus:

- **Linux:** `sudo apt-get install incus && sudo incus admin init --auto`.
  Your LAN interface is the one `ip route show default` names, such as `eno1`.
- **Mac:** with [Homebrew](https://brew.sh), run
  `brew install colima incus` and
  `colima start --runtime incus --network-mode bridged --network-interface en0`
  (`en0` is usually Wi-Fi; use your LAN interface). Your LAN interface inside
  Colima is `col0`. Drop `sudo` from the commands below.

1. Create an Incus container on your LAN, with `LAN` replaced by that
   interface:

   ```bash
   sudo incus init images:ubuntu/26.04 libertas-hub -c security.nesting=true
   sudo incus config device add libertas-hub eth0 nic nictype=macvlan parent=LAN
   sudo incus start libertas-hub && sleep 10
   ```

2. Install the Hub in it:

   ```bash
   sudo incus exec libertas-hub -- bash -c '
     apt-get update && apt-get install -y curl
     install -d -m 0755 /etc/apt/keyrings
     curl -fsSL https://apt.libertasiot.com/ubuntu/keys/libertasiot-archive-keyring.gpg \
       > /etc/apt/keyrings/libertasiot-archive-keyring.gpg
     echo "deb [signed-by=/etc/apt/keyrings/libertasiot-archive-keyring.gpg] https://apt.libertasiot.com/ubuntu resolute main" \
       > /etc/apt/sources.list.d/libertasiot.list
     apt-get update && apt-get install -y libertasd'
   ```

3. Find its address: `sudo incus list libertas-hub` shows the Hub's LAN
   address. Phones and other computers reach it; the computer running it
   cannot (a macvlan limit).

4. On the same Wi-Fi, tap **Add a new Hub** → **Search Local Network** in the
   App and choose the Hub. **Agree** to the EULA and create the administrator
   in **Setup Admin Account**. That password is the master password: write it
   down.

### Your own Cloud Hub on a server

It runs on an Ubuntu 26.04 server with a public address. Open TCP ports 8080
and 8443, and run these steps on the server itself.

1. Install the Hub:

   ```bash
   sudo install -d -m 0755 /etc/apt/keyrings
   curl -fsSL https://apt.libertasiot.com/ubuntu/keys/libertasiot-archive-keyring.gpg \
     | sudo tee /etc/apt/keyrings/libertasiot-archive-keyring.gpg >/dev/null
   echo 'deb [signed-by=/etc/apt/keyrings/libertasiot-archive-keyring.gpg] https://apt.libertasiot.com/ubuntu resolute main' \
     | sudo tee /etc/apt/sources.list.d/libertasiot.list >/dev/null
   sudo apt-get update && sudo apt-get install -y libertasd
   sudo mkdir -p /etc/systemd/system/libertasd.service.d
   printf '[Service]\nEnvironment=LIBERTAS_CLOUD_HUB=1\n' \
     | sudo tee /etc/systemd/system/libertasd.service.d/10-cloud-hub.conf >/dev/null
   sudo systemctl daemon-reload && sudo systemctl stop libertasd
   ```

2. Commission it with the address clients will use. The first run prints where
   to read the Beta EULA and the `--accept-eula` value; rerun with it. Type the
   master password when asked and write it down: it cannot be recovered.

   ```bash
   sudo env LIBERTAS_CLOUD_HUB=1 libertasd commission \
     --connection static --connection-address hub.example.com:8080
   sudo env LIBERTAS_CLOUD_HUB=1 libertasd commission --accept-eula <value> \
     --connection static --connection-address hub.example.com:8080
   sudo systemctl start libertasd
   ```

3. In the App, tap **Add a new Hub** → **By IP Address**, enter
   `hub.example.com:8080`, and log in as **Admin** with the master password.

## Studio

1. Install Node.js and Rust, then the [Libertas Studio](../studio.md) extension
   in VS Code; it checks its prerequisites on first launch.
2. Run **Libertas: Manage Hubs** and add the Hub by name and address
   (`192.168.1.50` for a Home Hub, `hub.example.com:8080` for your own Cloud Hub).
3. Studio shows **Authorize Libertas Hub** with a QR code. In the signed-in
   Libertas One app, use **Scan Login QR Code**.
4. Run **Libertas: Create New Project**, then follow
   [Developing Apps](developing-apps.md).

Next: [Using Studio](studio.md) · [Administration](administration.md) ·
[Troubleshooting](troubleshooting.md)
