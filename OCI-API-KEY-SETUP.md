# OCI API Key Setup — VM bina 2FA code ke banane ke liye

Goal: ek baar API key ban jaye, phir VM banana CLI se hoga — browser login aur 2FA code kabhi nahi chahiye. Capacity ke liye CLI se ek command me turant retry hota hai.

## Part 1 — Laptop pe OCI CLI install karo (Hermes / terminal)

```bash
bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)"
oci --version
```

Agar installer PATH me `oci` nahi laata, to naya terminal tab kholo ya `~/bin/oci --version` try karo.

## Part 2 — `oci setup config` chalao (laptop terminal)

```bash
oci setup config
```

Ye ye puchega — sab Oracle console se milega (Profile icon → Tenancy / User):
- Tenancy OCID — `ocid1.tenancy.oc1..aaaa...`
- User OCID — `ocid1.user.oc1..aaaa...`
- Region — `ap-mumbai-1`
- Private key path — default `~/.oci/oci_api_key.pem` rehne do
- Public key path — default `~/.oci/oci_api_key_public.pem` rehne do

Setup ek key pair banayega. Public key file ka content copy karo:
```bash
cat ~/.oci/oci_api_key_public.pem
```

Private key KABHI kisi ko mat bhejo, chat/Telegram me mat paste karo. Sirf PUBLIC key console me upload hoti hai.

## Part 3 — Public key Oracle console me upload karo (browser, ek baar code deke login karo)

1. cloud.oracle.com pe login karo (2FA code deke — ye aakhri baar code chahiye hoga).
2. Left menu: **Identity & Security → Users** → apna username click karo.
3. Neeche **API Keys** section → **Add API Key**.
4. **Paste Public Key** chuno, Part 2 wali public key paste karo → **Add**.
5. Fingerprint dikhega — usko `~/.oci/config` wali fingerprint se milao, same honi chahiye.

## Part 4 — Test karo (laptop terminal, bina code ke)

```bash
oci iam compartment list --compartment-id <TENANCY_OCID>
```

List aa gayi = API key kaam kar raha hai, ab code kabhi nahi chahiye.

## Part 5 — VM launch (CLI, capacity retry)

Pehle OCIDs nikalo (ek baar):
```bash
oci iam compartment list --compartment-id <TENANCY_OCID> --query 'data[*].{name:name, id:id}'
oci network vcn list --compartment-id <COMPARTMENT_OCID> --query 'data[*].{name:"display-name", id:id}'
oci network subnet list --compartment-id <COMPARTMENT_OCID> --vcn-id <VCN_OCID> --query 'data[*].{name:"display-name", id:id}'
oci compute image list --compartment-id <COMPARTMENT_OCID> --operating-system "Canonical Ubuntu" --operating-system-version "24.04" --shape VM.Standard.A1.Flex --query 'data[*].{name:"display-name", id:id, time: "time-created"}'
```

Phir launch (pehle 1 OCPU / 6 GB, capacity na mile to 4 OCPU / 24 GB):
```bash
oci compute instance launch \
  --compartment-id <COMPARTMENT_OCID> \
  --availability-domain "AD-1" \
  --shape VM.Standard.A1.Flex \
  --shape-config '{"ocpus":1,"memoryInGBs":6}' \
  --image-id <IMAGE_OCID> \
  --subnet-id <SUBNET_OCID> \
  --assign-public-ip true \
  --display-name videobot
```

Capacity error aaye to wahi command dobara chalao — CLI me dobara config bharne ki zaroorat nahi, isiliye browser se behtar hai.

## Notes
- Oracle 25D+ pe MFA enforced hai, disable nahi hota — isiliye API key hi sahi rasta hai.
- VCN `videobot-vcn` (10.0.0.0/16) + public subnet pehle se bana hai, dobara banane ki zaroorat nahi.
- Morning 06:19 browser retry cron tab tak backup rehne do, API key test pass hote hi use hata denge.
