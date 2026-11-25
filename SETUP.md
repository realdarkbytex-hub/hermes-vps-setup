# Run Hermes 24/7 — Your Own AI: Full Setup

Goal: your own Hermes AI agent running 24/7 on an Oracle Cloud Always Free VPS, reachable from Telegram, so it works when your laptop is off.

## Part 1 — OCI API key (do this once, never need a 2FA code again)

Oracle 25D+ enforces MFA and it can't be disabled. Browser VM retries will keep dying at the authenticator code. An OCI API key signs CLI requests — no interactive login, no 2FA.

1. Install the OCI CLI on your laptop:
   ```bash
   bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)"
   oci --version
   ```
2. Run `oci setup config`. It asks for:
   - Tenancy OCID and User OCID (Oracle console → Profile icon → Tenancy / User)
   - Region: `ap-mumbai-1`
   - Key pair paths — accept the defaults
   - **Passphrase: type the literal text `N/A`** (pressing Enter empty creates an encrypted key and the CLI will keep asking for a passphrase)
3. Upload the PUBLIC key to the console: Identity & Security → Users → your user → API Keys → Add API Key → Paste Public Key. Only the public key goes in the console — the private key never leaves your machine, never paste it in chat.
4. Test (no code needed):
   ```bash
   oci iam compartment list --compartment-id <TENANCY_OCID>
   ```
   A list = the key works.

## Part 2 — Launch the VM (CLI, with capacity retries)

- Shape: `VM.Standard.A1.Flex`, 1 OCPU / 6 GB RAM, Always Free
- Image: Canonical Ubuntu 24.04 aarch64 (latest)
- Region: `ap-mumbai-1`, Availability Domain: AD-1 (usually the only AD in a tenancy)
- VCN + public subnet: create once, reuse

```bash
oci compute instance launch \
  --compartment-id <COMPARTMENT_OCID> \
  --availability-domain "<AD-1-FULL-NAME>" \
  --shape VM.Standard.A1.Flex \
  --shape-config '{"ocpus":1,"memoryInGBs":6}' \
  --image-id <IMAGE_OCID> \
  --subnet-id <SUBNET_OCID> \
  --assign-public-ip true \
  --ssh-authorized-keys-file ~/.ssh/<your-key>.pub \
  --display-name <name>
```

**Launch WITH `--ssh-authorized-keys-file`.** This is the single most important flag. Oracle has no way to add an SSH key to a launched instance — no console option, no `user_data` on a launched instance, and the Run Command agent runs as `ocarun` with no sudo and no write access to `/home/ubuntu`. Launching without a key means a GRUB `init=/bin/bash` recovery over VNC + serial console.

**Capacity:** "Out of host capacity" is normal for Always Free Ampere. Retry the same command every 15 minutes (a background loop works). Do NOT terminate a running VM to retry — you may not get one back.

## Part 3 — Install Hermes on the VM

SSH in as `ubuntu`, then install Hermes the same way as on the laptop (v0.23.4, aarch64 build). Then:

1. Copy the workflow skill to the VM (`noya-owns-workflow` — see `WORKFLOW.md`).
2. Set the model to **direct Gemini** (the VM has no OmniRoute):
   - Primary: `gemini-3.1-flash-lite`
   - Fallback: NVIDIA NIM, `nvidia/nemotron-3.5-super-vl-120b-a12b`, reasoning effort `high` (never `medium` — NVIDIA rejects it with 400)
   - Copy the API keys from your laptop Hermes config (`~/.hermes/config.yaml`, `~/.hermes/.env`) directly to the VM — never paste a key or token in chat.
3. Set up the Telegram gateway on the VM with your bot token (same token as the laptop gateway). Allowlist ONLY your own Telegram ID.
4. Install the gateway as a systemd service so it survives reboots.

## Part 4 — Move the gateway (one poller only)

**Stop the laptop gateway BEFORE starting the VPS gateway.** One bot token = one poller. Two gateways polling the same token = a polling conflict, and Telegram delivers each update to only one of them.

1. Stop the laptop gateway.
2. Start the VPS gateway (`hermes gateway restart` on the VM).
3. Test: send `hi` to your bot. Confirm the reply comes from the VPS, not the laptop — ask Hermes "are you running on the VPS or the laptop?" and check its answer. A fresh VPS Hermes greets generically; the laptop one knows your conversation context.

## Part 5 — Verify

- `systemctl status hermes` (or your gateway service) — active
- Send `/start` then a real task to the bot — it should reply from the VPS
- Turn the laptop OFF and send another message — Hermes should still reply. That's the whole point.

## If you launched without an SSH key (recovery)

Don't terminate the VM. Recovery path that works:

1. Create a **serial console connection** (Compute → Instances → your VM → OS Management → Console connection → Create local connection). It needs an **RSA** key — Oracle rejects ed25519 for console connections.
2. Connect via VNC, not serial: SSH-tunnel port 5900 to the instance (`-L localhost:5900:<INSTANCE-OCID>:5900`), then `vncviewer localhost:5900`.
3. Reset the VM, and **hold Esc from the moment the screen goes black** until the GRUB menu (or `grub>` prompt) appears. GRUB is graphical-only — it never prints to the serial console, so don't waste time there.
4. At the GRUB prompt, boot with `init=/bin/bash` (find the real `/boot` partition first — on this image it's not the first partition; `ls` each one and look for `vmlinuz-*`):
   ```
   set root=(hd0,gpt16)
   linux /vmlinuz-7.0.0-1017-oracle root=/dev/sda1 ro init=/bin/bash
   initrd /initrd.img-7.0.0-1017-oracle
   boot
   ```
   (Partition numbers and kernel versions vary — check with `ls` first.)
5. The root shell talks to the **serial** console, not VNC — VNC will stay black. Open a serial-console SSH session from an interactive terminal to see the root shell.
6. In the root shell: `mount -o remount,rw /`, append your public keys to `/home/ubuntu/.ssh/authorized_keys`, `chmod 700 /home/ubuntu/.ssh`, `chmod 600 /home/ubuntu/.ssh/authorized_keys`, `chown -R ubuntu:ubuntu /home/ubuntu/.ssh`, `sync; reboot -f`.
7. Test: `ssh -i ~/.ssh/<key> ubuntu@<VM-IP> "echo ok"` should print `ok`.
