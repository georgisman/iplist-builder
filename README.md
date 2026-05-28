# IP List Builder

Static GitHub Pages mini-site for building domain, IPv4, CIDR and Keenetic BAT route lists from predefined service groups.

The site has no backend. DNS resolving and generated file creation happen in GitHub Actions. The frontend only reads files from `generated/` and combines selected groups in the browser.

## Project structure

```text
/
├── index.html
├── style.css
├── app.js
├── .nojekyll
├── sources/services.json
├── generated/
├── scripts/generate.py
├── scripts/requirements.txt
└── .github/workflows/update-lists.yml
```

## Groups

Groups are defined in `sources/services.json`:

```json
{
  "id": "vpn",
  "title": "VPN / proxy / privacy tunnels",
  "description": "VPN, proxy and censorship-circumvention services",
  "domains": ["mullvad.net", "protonvpn.com"]
}
```

Domains are normalized to lowercase and deduplicated. Entries that start with a dot, for example `.ua`, are treated as TLD markers. They are included in domain lists but are not resolved to IPv4/CIDR.

## Updating lists manually

Open the repository on GitHub, go to **Actions**, choose **update-lists**, then run **Run workflow**.

The workflow also runs automatically every 12 hours.

## GitHub Pages

Enable Pages in repository settings:

1. Open **Settings → Pages**.
2. Select **Deploy from a branch**.
3. Select your default branch, usually `main`.
4. Select `/root` as the publishing folder.
5. Save.

The `.nojekyll` file disables Jekyll processing for this static project.

## Add new domains

Edit `sources/services.json`, add domains to the needed group, commit and run the workflow. Do not add protocol prefixes like `https://`. Use plain domains only.

## Keenetic BAT

Keenetic BAT is generated from CIDR entries. Each route line looks like this:

```bat
route add 1.1.1.1 mask 255.255.255.255 0.0.0.0
```

The frontend can combine several selected groups and generate one BAT file from merged CIDR values.

## Safe-mode CIDR warning

CIDR generation is conservative. Each IPv4 address is treated as `/32`, then adjacent addresses are collapsed with `ipaddress.collapse_addresses`. The generator does not aggressively expand to `/24`, does not pull ASN networks and does not use whois/BGP. This avoids grabbing unrelated CDN networks, but it also means the list may not cover every IP used by a service.

## Local run

```bash
pip install -r scripts/requirements.txt
python scripts/generate.py
python3 -m http.server 8000
```

Open `http://localhost:8000`.
