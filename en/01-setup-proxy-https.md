# 01. Setup, Proxy & CA Certificate Configuration

> 🌐 **Language / اللغة:** [النسخة العربية (`ar/01-setup-proxy-https.md`)](../ar/01-setup-proxy-https.md)

---

## 🎯 Chapter Objectives

Establish an operational testing environment for **Burp Suite**, understand the underlying networking mechanics of intercepting HTTP/HTTPS proxies as a *Man-in-the-Middle*, configure external browsers (Firefox / Chromium) via `FoxyProxy`, and install the `PortSwigger Root CA Certificate` to seamlessly decrypt and inspect TLS-encrypted communications without certificate warnings.

---

## 🔍 Technical Explanation

Burp Suite's **Proxy** tab functions as a local HTTP/S interception daemon positioned directly in the communication channel between the client browser and the upstream web server, listening by default on loopback address `127.0.0.1` and TCP port `8080`.

```
[ Browser (Client) ] <---> [ Burp Suite Proxy (127.0.0.1:8080) ] <---> [ Web Server ]
```

### The HTTPS Dilemma and TLS Decryption (SSL Termination):
Under standard HTTPS operation, the browser negotiates an end-to-end encrypted TLS tunnel directly with the remote web server, preventing any intermediary from reading or modifying the payload in transit.
To inspect, intercept, and rewrite encrypted requests and responses, Burp Suite acts as a transparent **Man-in-the-Middle (MitM)**:

1. **Interception of the CONNECT Tunnel:** When navigating to a secure origin (e.g., `https://example.com`), the browser submits an HTTP `CONNECT example.com:443` request to Burp's listening interface.
2. **Dynamic On-the-Fly Certificate Synthesis:** Burp dynamically generates a host certificate specifically for `example.com`, cryptographically signed by its own internal certificate authority (`PortSwigger CA`).
3. **Trust Chain Verification:** If `PortSwigger CA` has not been installed into the browser's or operating system's Trusted Root Certification Authorities store, the browser strictly aborts the connection with a fatal cryptographic warning (`SEC_ERROR_UNKNOWN_ISSUER`).
4. **Independent TLS Sessions:** Once trusted, Burp cleanly terminates the incoming TLS handshake from the browser, inspects and modifies the plaintext data within the GUI, and initiates a completely independent outbound TLS handshake with the real destination server.

---

## 🛠️ Step-by-Step Hands-On Guide

### Step 1: Downloading & Initializing Burp Suite
1. Download the installer matching your OS from the official [PortSwigger Releases](https://portswigger.net/burp/releases).
2. Launch Burp Suite:
   * Select **Temporary project in memory** (Community) or **New project on disk** (Professional).
   * Accept defaults by choosing **Use Burp defaults** and click **Start Burp**.

### Step 2: Configuring Firefox with FoxyProxy
1. In Mozilla Firefox, install the **FoxyProxy Standard** extension from the official Firefox Add-ons store.
2. Click the FoxyProxy icon in the toolbar > select **Options** > click **Add**.
3. Input the listener coordinates:
   * **Title / Description:** `Burp Suite`
   * **Proxy Type:** `HTTP`
   * **Proxy IP address or DNS name:** `127.0.0.1`
   * **Port:** `8080`
4. Click **Save**, then activate the newly created `Burp Suite` profile from the extension drop-down.

### Step 3: Exporting and Trusting the Root CA Certificate
1. Ensure Burp is running and FoxyProxy is set to route through Burp.
2. Open a new browser tab and navigate to `http://burp` (or `http://127.0.0.1:8080`).
3. In the upper-right corner of the welcome screen, click **CA Certificate** to download `cacert.der`.
4. Inside Firefox Settings:
   * Navigate to **Settings** > search for `Certificates` > click **View Certificates**.
   * Under the **Authorities** tab, click **Import**.
   * Select the downloaded `cacert.der` file.
   * **Critical Step:** Check the checkbox for: **Trust this CA to identify websites**, then click **OK**.

---

## 🧪 Recommended Labs

* **Foundational Lab:** [TryHackMe: Burp Suite - The Basics (Tasks 5 & 6)](https://tryhackme.com/room/burpsuitebasics)
  * Comprehensive walk-through on browser routing, root trust configuration, and intercepting your first live HTTP transaction.

---

## ⚔️ Attacker vs. 🛡️ Defender Perspective

* **⚔️ Attacker Perspective:**
  * The proxy reveals ground truth by dismantling the browser abstraction layer. Any client-side restriction (character blacklists, hidden inputs, disabled buttons, client-side encryption) is immediately exposed as cosmetic and easily modifiable.
* **🛡️ Defender Perspective:**
  * Understanding interception proxies teaches software engineers that the client environment must be treated as hostile and untrusted. All data validation, authentication boundaries, and permission checks must reside exclusively on the server side.

---

## ❌ Common Beginner Mistakes

> [!WARNING]
> 1. **Failing to check the "Trust to identify websites" checkbox:** Simply importing the certificate without explicitly enabling website identification trust will result in persistent TLS errors on all HTTPS targets.
> 2. **Port 8080 conflicts:** If an existing local daemon (e.g., Apache, Tomcat, or Docker) binds to port 8080, Burp's listener will fail to start. Resolve this under `Proxy > Proxy settings > Proxy listeners` by switching the port to `8081` or `8888` and updating FoxyProxy accordingly.
> 3. **Leaving the proxy active during regular browsing:** Causes internet access to fail whenever Burp is closed and clutters your engagement project with personal credentials and third-party tracking traffic.

---

## 📚 References

1. [PortSwigger Documentation: Installing CA Certificates](https://portswigger.net/burp/documentation/desktop/external-browser-config/certificate/firefox)
2. [Mozilla Developer Network: TLS/SSL Certificate Chains](https://developer.mozilla.org/en-US/docs/Glossary/Certificate)
3. [FoxyProxy Official User Guide](https://getfoxyproxy.org/)

---

| [← Previous: 00. Editions Comparison](./00-editions-comparison.md) | [TOC](../README.en.md#🗂️-table-of-contents-english-version---en) | [Next: 02. Dashboard & Scope →](./02-dashboard-target-scope.md) |
