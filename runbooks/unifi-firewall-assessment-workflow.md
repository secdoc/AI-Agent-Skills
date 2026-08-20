<!--
Licensed under CC BY 4.0 (see ../LICENSE-docs and ../NOTICE).
Attribution: Lester E. Nichols III, secdoc.tech.
Anonymized reference runbook. All IPs are RFC 5737 documentation ranges;
substitute your own environment's real values.
-->
# UniFi Cloud Gateway: Firewall Assessment and Change Workflow

**Document status:** Final (drafted, then reviewed for accuracy and anonymization)  
**Applies to:** UniFi OS / Network 9.x with Zone-Based Firewall (ZBF)  
**Audience:** Technicians with general Linux/terminal comfort but no prior UniFi or firewall API experience

> **Anonymization note for the reader.** Every IP address in this document (`192.0.2.x`) is from a reserved documentation range and is not a real host. Wherever you see `<GATEWAY_IP>`, a `<...>` placeholder, or a `192.0.2.x` address, substitute your own environment's real value. The API key is never written into any command; it is always read from an environment variable you set in Phase 1.

---

## Read This First

Every command in this runbook that writes to the gateway changes live network behavior immediately. There is no "staging" mode. A misconfigured firewall rule can silently cut legitimate traffic, isolate production systems, or create security holes, and the gateway will not warn you before applying the change. Before you touch Phase 4, make sure you have completed the snapshot in Phase 2 and have a rollback plan for every change you intend to make. Work one change at a time. If anything unexpected happens (rising BLOCK hit counters, reachability loss, alerts from monitoring), stop and roll back before diagnosing. The sections below walk you through each stage in order. Do not skip ahead.

---

## Glossary

**API key:** A long random string that proves your identity to the gateway's software interface. It works like a password specific to one admin account. You pass it in every request so the gateway knows who is asking.

**Endpoint:** A specific URL path on the gateway that returns or accepts a particular type of data. For example, `/firewall-policies` is the endpoint for firewall rules. Think of it as a door into a specific room in the gateway's configuration.

**Zone:** A named group of networks (VLANs, interfaces) that the firewall treats as a single trust boundary. Traffic flowing from one zone to another is subject to inter-zone firewall rules. Traffic within the same zone is typically unrestricted unless you configure intra-zone rules.

**ZBF (Zone-Based Firewall):** The firewall model used in UniFi Network 9.x. Instead of writing rules per-interface, you assign interfaces to zones and write policies between zones. This is more scalable and easier to reason about than older per-interface rulesets.

**VLAN (Virtual LAN):** A logical network segment. A single physical switch can carry many VLANs, each isolated from the others at Layer 2. VLANs are how most UniFi environments divide traffic (for example: VLAN 10 for workstations, VLAN 20 for IoT, VLAN 30 for cameras).

**Policy:** A single firewall rule in ZBF. Each policy has a source zone, destination zone, matching criteria (IP, port, protocol), and an action (ALLOW or BLOCK). Policies in ZBF are ordered and evaluated top-to-bottom within each zone pair.

**First-match:** The firewall stops evaluating rules as soon as one rule matches a packet. If a BLOCK rule appears above an ALLOW rule for the same traffic, the traffic is blocked even though the ALLOW rule exists further down. Order matters critically.

---

## Prerequisites

- A workstation with `curl`, `jq`, and `python3` installed (any modern Linux or macOS machine qualifies).
- Network access to the gateway's management IP from your workstation.
- An admin account on the UniFi gateway (you will create an API key under it in Phase 1).
- A text editor and a directory where you can write files.

---

## Phase 1: API Key Setup and Connectivity Check

### What this does and why it matters

Before you can read or write anything on the gateway via the API, you need an API key. This phase walks you through creating one inside the UniFi web UI and verifying that your workstation can reach the gateway API and authenticate successfully. If this probe fails, nothing in the later phases will work, so there is no point continuing until this is clean.

For read-only assessment work, create the API key under a dedicated View Only admin account rather than your full admin account. This limits the blast radius if the key is ever exposed: a View Only key cannot make changes, so an attacker who obtains it cannot alter your firewall.

### Steps

1. Open a browser and navigate to your gateway's web UI:

   ```
   https://<GATEWAY_IP>
   ```

   Replace `<GATEWAY_IP>` with the actual management IP of your UniFi Cloud Gateway. If you are unsure of the IP, check your DHCP server, or look it up in the UniFi console. An example IP from the documentation-reserved range is `192.0.2.1`.

2. Log in with your admin credentials.

3. Navigate to the API key creation screen. On Network 9.x this is in one of two places depending on your exact build:

   - **Option A (most 9.x builds):** Settings > Control Plane > Integrations > API Keys
   - **Option B (some 9.x builds):** Admins and Users > (click your admin user) > Create API Key

   If you want a read-only key (recommended for assessment), first create a View Only admin under Admins and Users, log in as that admin, then create the API key there.

4. Click **Create API Key**, give it a descriptive name (for example: `firewall-assessment-2024`), and copy the key value immediately. The UI will not show it again after you close the dialog.

5. On your workstation, create a file to hold the key. Never put the raw key in a script or command history:

   ```bash
   mkdir -p ~/unifi-config
   chmod 700 ~/unifi-config
   cat > ~/unifi-config/.env << 'EOF'
   export UNIFI_API_KEY="PASTE_YOUR_KEY_HERE"
   export UNIFI_GW="<GATEWAY_IP>"
   EOF
   chmod 600 ~/unifi-config/.env
   ```

   Replace `PASTE_YOUR_KEY_HERE` with the key you copied in step 4, and replace `<GATEWAY_IP>` with your gateway's IP.

6. Load the variables into your shell session:

   ```bash
   source ~/unifi-config/.env
   ```

   You must repeat this step every time you open a new terminal. The variables are not permanent by default.

7. Run the connectivity and authentication probe:

   ```bash
   curl -sk "https://${UNIFI_GW}/proxy/network/integration/v1/sites" \
     -H "X-API-Key: $UNIFI_API_KEY" | python3 -m json.tool
   ```

   The flags mean: `-s` = silent (no progress bar), `-k` = accept the gateway's self-signed TLS certificate. The gateway ships with a self-signed certificate, so `-k` is required unless you have installed a trusted certificate.

8. Note the site ID from the response. It looks like a UUID (for example: `abc12345-abcd-1234-abcd-1234567890ab`). Most single-gateway setups have one site named `default`. Subsequent endpoints in this runbook use `default` as the site name; if your site has a different name, substitute it.

### Expected result

The command returns a JSON array containing at least one site object, for example:

```json
[
  {
    "_id": "abc12345-abcd-1234-abcd-1234567890ab",
    "name": "default",
    ...
  }
]
```

HTTP status 200 with a site list means your key is valid and your workstation can reach the API.

### If it goes wrong

| Symptom | Likely cause | Fix |
|---|---|---|
| `curl: (7) Failed to connect` | Workstation cannot reach the gateway IP | Check routing, VPN, firewall between you and the gateway |
| `curl: (60) SSL certificate problem` | You forgot `-k` | Add `-k` to the curl command |
| HTTP 401 Unauthorized | Wrong or missing API key | Re-check the key in `~/unifi-config/.env`; reload with `source` |
| HTTP 403 Forbidden | Key exists but the account lacks permission | Verify the admin account used to create the key has at least View Only access |
| Empty JSON `[]` | Key is valid but no sites found | Check gateway is running Network 9.x and that the Control Plane is active |

---

## Phase 2: Read-Only Full Config Pull (Snapshot)

### What this does and why it matters

Before touching anything, capture the full current state of the gateway to a local directory. This serves two purposes. First, it gives you the data you need for assessment without having to re-query the API repeatedly. Second, it is your rollback reference: if a change in Phase 4 breaks something, you have the exact pre-change object in a local file and can PUT it back immediately.

Pull everything to files. Do not try to evaluate the config live in your head from curl output.

### Steps

1. Create an output directory with today's date and time in the name so you can distinguish snapshots:

   ```bash
   SNAPSHOT_DIR=~/unifi-config/snapshots/$(date +%Y%m%d-%H%M%S)
   mkdir -p "$SNAPSHOT_DIR"
   echo "Snapshot directory: $SNAPSHOT_DIR"
   ```

2. Set common URL prefixes to reduce typing:

   ```bash
   V2="https://${UNIFI_GW}/proxy/network/v2/api/site/default"
   CL="https://${UNIFI_GW}/proxy/network/api/s/default"
   HDR="-H \"X-API-Key: $UNIFI_API_KEY\""
   ```

   Note: the `HDR` variable is a convenience label for the documentation below. In actual curl commands, pass the header inline as shown.

3. Pull all ZBF-related data from the v2 API. Each command writes one JSON file:

   ```bash
   # ZBF firewall policies (the actual rules)
   curl -sk "${V2}/firewall-policies" \
     -H "X-API-Key: $UNIFI_API_KEY" \
     > "${SNAPSHOT_DIR}/zbf-policies.json"

   # Zone definitions
   curl -sk "${V2}/firewall/zone" \
     -H "X-API-Key: $UNIFI_API_KEY" \
     > "${SNAPSHOT_DIR}/zbf-zones.json"

   # Zone-to-zone action matrix (default actions between each zone pair)
   curl -sk "${V2}/firewall/zone-matrix" \
     -H "X-API-Key: $UNIFI_API_KEY" \
     > "${SNAPSHOT_DIR}/zbf-zone-matrix.json"

   # Traffic rules (application-level rules, separate from ZBF policies)
   curl -sk "${V2}/trafficrules" \
     -H "X-API-Key: $UNIFI_API_KEY" \
     > "${SNAPSHOT_DIR}/traffic-rules.json"

   # Traffic routes
   curl -sk "${V2}/trafficroutes" \
     -H "X-API-Key: $UNIFI_API_KEY" \
     > "${SNAPSHOT_DIR}/traffic-routes.json"
   ```

4. Pull classic API data (these endpoints use the older API path but work fine with the same API key):

   ```bash
   # Firewall address groups (used for grouping IPs in ZBF policies)
   curl -sk "${CL}/rest/firewallgroup" \
     -H "X-API-Key: $UNIFI_API_KEY" \
     > "${SNAPSHOT_DIR}/firewall-groups.json"

   # Port forwarding rules
   curl -sk "${CL}/rest/portforward" \
     -H "X-API-Key: $UNIFI_API_KEY" \
     > "${SNAPSHOT_DIR}/port-forwards.json"

   # Network and VLAN configuration
   curl -sk "${CL}/rest/networkconf" \
     -H "X-API-Key: $UNIFI_API_KEY" \
     > "${SNAPSHOT_DIR}/networks.json"

   # Static routes
   curl -sk "${CL}/rest/routing" \
     -H "X-API-Key: $UNIFI_API_KEY" \
     > "${SNAPSHOT_DIR}/static-routes.json"

   # WLAN (wireless network) configuration
   curl -sk "${CL}/rest/wlanconf" \
     -H "X-API-Key: $UNIFI_API_KEY" \
     > "${SNAPSHOT_DIR}/wlans.json"

   # All gateway settings (includes IPS/threat management state)
   curl -sk "${CL}/get/setting" \
     -H "X-API-Key: $UNIFI_API_KEY" \
     > "${SNAPSHOT_DIR}/settings.json"

   # Basic device inventory
   curl -sk "${CL}/stat/device-basic" \
     -H "X-API-Key: $UNIFI_API_KEY" \
     > "${SNAPSHOT_DIR}/devices.json"

   # Site health summary
   curl -sk "${CL}/stat/health" \
     -H "X-API-Key: $UNIFI_API_KEY" \
     > "${SNAPSHOT_DIR}/health.json"

   # System info
   curl -sk "${CL}/stat/sysinfo" \
     -H "X-API-Key: $UNIFI_API_KEY" \
     > "${SNAPSHOT_DIR}/sysinfo.json"
   ```

5. Verify all files were written and contain non-empty JSON:

   ```bash
   for f in "${SNAPSHOT_DIR}"/*.json; do
     size=$(wc -c < "$f")
     first_char=$(head -c 1 "$f")
     if [ "$size" -lt 5 ] || [ "$first_char" = "<" ]; then
       echo "WARN: $f may be empty or returned HTML (check manually)"
     else
       echo "OK: $f ($size bytes)"
     fi
   done
   ```

   The check `first_char = "<"` catches cases where the gateway returned an HTML error page instead of JSON, which can happen if the URL path is wrong.

6. Count the firewall policies so you know what you are working with:

   ```bash
   jq '.data | length' "${SNAPSHOT_DIR}/zbf-policies.json" 2>/dev/null \
     || jq 'length' "${SNAPSHOT_DIR}/zbf-policies.json"
   ```

   Note: Some endpoints wrap results in `{"data": [...]}` and some return a bare array. The command above tries both forms.

7. Record the snapshot path in a log file you will use throughout this workflow:

   ```bash
   echo "$(date -Iseconds) SNAPSHOT path=${SNAPSHOT_DIR}" \
     >> ~/unifi-config/change-log.txt
   ```

### Expected result

Each `.json` file in `$SNAPSHOT_DIR` should be valid JSON starting with `{` or `[`. The `zbf-policies.json` file on an active gateway typically contains 100 to 400+ policies (many are predefined by UniFi). The `zbf-zones.json` file will contain the zones configured on the gateway. All other files should have recognizable content when you run `jq '.' <file> | head -40`.

### If it goes wrong

**A file starts with `<` (HTML):** The endpoint URL is wrong. Double-check the path against the endpoint reference table in Phase 3.

**A file is completely empty (0 bytes):** The curl command failed silently. Re-run just that command without `-s` to see the error:

```bash
curl -k "https://${UNIFI_GW}/proxy/network/v2/api/site/default/firewall-policies" \
  -H "X-API-Key: $UNIFI_API_KEY"
```

**`jq` reports `null` or parse errors:** The response might be a JSON error object from the gateway. Inspect the file directly: `cat "${SNAPSHOT_DIR}/zbf-policies.json"`. If the gateway returned `{"errorCode": "InvalidObject"}`, you are hitting a classic-API path that does not exist in 9.x ZBF mode. Use the v2 paths listed in step 3.

---

## Phase 3: Assessment and Evaluation

### What this does and why it matters

You now have the full gateway config on disk. This phase is entirely read-only: you analyze the JSON files, take notes, and build a list of findings. Nothing you do here affects the running gateway. Take your time. Good assessment findings drive good changes, and bad findings drive unnecessary or dangerous changes.

The goal is to understand: what zones exist, what networks are in each zone, what the default inter-zone actions are, which custom policies exist and why, and where the security posture is weaker than intended.

### API Endpoint Reference

| Data | API version | Endpoint path (append to base URL) | Response wrapper |
|---|---|---|---|
| ZBF policies | v2 | `/proxy/network/v2/api/site/default/firewall-policies` | `{data:[...]}` or bare array |
| Zone definitions | v2 | `/proxy/network/v2/api/site/default/firewall/zone` | bare array |
| Zone matrix | v2 | `/proxy/network/v2/api/site/default/firewall/zone-matrix` | object |
| Traffic rules | v2 | `/proxy/network/v2/api/site/default/trafficrules` | object |
| Traffic routes | v2 | `/proxy/network/v2/api/site/default/trafficroutes` | object |
| Firewall groups | classic | `/proxy/network/api/s/default/rest/firewallgroup` | `{meta,data:[...]}` |
| Port forwards | classic | `/proxy/network/api/s/default/rest/portforward` | `{meta,data:[...]}` |
| Networks / VLANs | classic | `/proxy/network/api/s/default/rest/networkconf` | `{meta,data:[...]}` |
| Static routes | classic | `/proxy/network/api/s/default/rest/routing` | `{meta,data:[...]}` |
| WLANs | classic | `/proxy/network/api/s/default/rest/wlanconf` | `{meta,data:[...]}` |
| All settings / IPS | classic | `/proxy/network/api/s/default/get/setting` | `{meta,data:[...]}` |
| Device inventory | classic | `/proxy/network/api/s/default/stat/device-basic` | `{meta,data:[...]}` |
| Health | classic | `/proxy/network/api/s/default/stat/health` | `{meta,data:[...]}` |

**Important:** The following paths return errors or empty results on Network 9.x ZBF gateways even though they exist in documentation for older firmware. Do not use them: `/proxy/network/api/s/default/rest/firewallpolicies`, `/rest/firewallzone`, `/proxy/network/api/s/default/rest/firewallrule`.

### Steps

Work through these checks in order. For each check, write your findings in a plain text or markdown file (for example `~/unifi-config/assessment-findings.md`). Reference the relevant object IDs so you can act on them in Phase 4.

**Check 1: Understand the zone structure**

```bash
jq '.[] | {name: .name, id: ._id, network_count: (.network_ids | length)}' \
  "${SNAPSHOT_DIR}/zbf-zones.json"
```

Note which zones exist. The UniFi defaults are typically: Internal (LAN), External (WAN), Gateway (the gateway device itself), and possibly DMZ, Management, Cameras, or others you or a previous admin created. Pay attention to any zone where `network_count` is 0: a zone with no member networks is defined but contains nothing, meaning no traffic is actually subject to its policies. This is a finding worth noting.

**Check 2: Review the zone matrix (default inter-zone actions)**

```bash
jq '.' "${SNAPSHOT_DIR}/zbf-zone-matrix.json"
```

The zone matrix defines what happens to traffic between each pair of zones when no specific policy matches. The typical secure default is:

- Internal to External: ALLOW (users can reach the internet)
- External to Internal: BLOCK (internet cannot initiate connections in)
- Internal to Internal: depends on design (zone isolation vs. flat LAN)

If you see zone pairs with a default of ALLOW where you would expect BLOCK (for example, External to Management), note that as a finding.

**Check 3: Summarize firewall policies by action and status**

Policy counts on busy gateways run large (400+ total, roughly 145 of which are predefined by UniFi). Read individual rules only after you understand the summary.

```bash
# How many policies total?
jq '[.[] ] | length' "${SNAPSHOT_DIR}/zbf-policies.json" 2>/dev/null \
  || jq '.data | length' "${SNAPSHOT_DIR}/zbf-policies.json"

# Count by action (ALLOW vs BLOCK)
jq 'if type == "array" then . else .data end
    | group_by(.action)
    | map({action: .[0].action, count: length})' \
  "${SNAPSHOT_DIR}/zbf-policies.json"

# Count custom (non-predefined) policies only
jq 'if type == "array" then . else .data end
    | [.[] | select(.predefined != true)] | length' \
  "${SNAPSHOT_DIR}/zbf-policies.json"

# List disabled custom policies (candidates for cleanup)
jq 'if type == "array" then . else .data end
    | [.[] | select(.predefined != true and .enabled == false)]
    | .[] | {name: .name, id: ._id, action: .action}' \
  "${SNAPSHOT_DIR}/zbf-policies.json"
```

**Check 4: Find overly broad ALLOW policies**

An ALLOW policy with `source = ANY` and `destination = ANY` across a sensitive zone boundary is a red flag. Look for these:

```bash
jq 'if type == "array" then . else .data end
    | [.[] | select(
        .action == "ALLOW"
        and .predefined != true
        and (
          (.src.matching_target == "ANY" or .dst.matching_target == "ANY")
        )
      )]
    | .[] | {name: .name, id: ._id, src_zone: .src_zone, dst_zone: .dst_zone,
              src_target: .src.matching_target, dst_target: .dst.matching_target}' \
  "${SNAPSHOT_DIR}/zbf-policies.json"
```

For each result, ask: should traffic really be allowed from anywhere in the source zone to anywhere in the destination zone? Is this intentional or is it a lazy rule that accumulated over time?

**Check 5: Look for port forwards without matching ZBF allow rules**

A port forward (NAT) alone does not guarantee that the forwarded traffic is permitted through the ZBF. If a port forward exists but no ZBF ALLOW policy covers the same traffic from External to the target host's zone, the traffic will be dropped by the firewall after the NAT translation.

```bash
# List all port forwards
jq '.data[] | {name: .name, dst_port: .dst_port, fwd: .fwd, proto: .proto, enabled: .enabled}' \
  "${SNAPSHOT_DIR}/port-forwards.json"
```

For each enabled port forward, find the destination IP (the `fwd` field) and determine which zone that IP lives in. Then look for a corresponding ZBF ALLOW policy from External to that zone covering the same port and protocol. If no such policy exists, the port forward is effectively broken (or the zone matrix default from External to that zone is already ALLOW, which may be too permissive).

**Check 6: Review IPS / threat management coverage**

```bash
jq '.data[] | select(.key == "ips") | {enabled: .enabled, mode: .ips_mode, categories: .ips_alert_categories}' \
  "${SNAPSHOT_DIR}/settings.json"
```

Note whether IPS is enabled and in what mode (detect-only vs. prevention). Note which zone interfaces are covered.

**Check 7: Review network and VLAN assignments**

```bash
jq '.data[] | {name: .name, vlan_id: .vlan, purpose: .purpose, subnet: .ip_subnet, enabled: .enabled}' \
  "${SNAPSHOT_DIR}/networks.json"
```

For each VLAN, note the `purpose` field. Understand that `purpose` is metadata only; it does not change gateway behavior. A VLAN labeled `guest` in the purpose field but placed in the Internal zone behaves like an internal network, not a guest network. Verify zone membership for each VLAN by cross-referencing `zbf-zones.json`.

**Check 8: Flag empty zones**

Any zone with `network_ids` empty (`[]`) was created but has no members. Policies targeting that zone have no effect on actual traffic. These are housekeeping findings.

```bash
jq '.[] | select((.network_ids | length) == 0) | {name: .name, id: ._id}' \
  "${SNAPSHOT_DIR}/zbf-zones.json"
```

**Check 9: Document inter-VLAN reachability vs. intent**

For each pair of VLANs, answer: should hosts on VLAN A be able to initiate connections to hosts on VLAN B? Write this down as a matrix. Compare it against the zone matrix and the custom policies. Gaps between intent and config are your primary findings.

**Compile your findings**

Write up every finding in `~/unifi-config/assessment-findings.md`. For each finding include:

- Description of the issue
- Evidence (the relevant JSON field and value, the policy ID or zone name)
- Risk rating (Low / Medium / High)
- Proposed remediation

Do not start Phase 4 until you have written up findings and agreed on a prioritized list of changes with whoever owns the environment.

### Expected result

You have a written findings document with at least the following confirmed: zone list, zone matrix default actions, custom policy count and summary, any overly broad ALLOW rules, any port forwards without matching ZBF policies, IPS state, and VLAN-to-zone mapping.

### If it goes wrong

**`jq` output says `null` everywhere:** The file has a response wrapper you did not account for. Check the structure with `jq 'keys' "${SNAPSHOT_DIR}/zbf-policies.json"`. If the result is `["data","meta"]`, your data is at `.data`. If it is `[]` or a number, you have a bare array.

**Zone matrix is an empty object `{}`:** The ZBF may not be enabled on this gateway, or the endpoint returned nothing. Check the gateway UI under Settings > Security > Zone-Based Firewall to confirm ZBF is active.

---

## Phase 4: The Change-Control Loop

### What this does and why it matters

This phase is where you make changes to the live gateway. Every step follows the same disciplined loop: propose in writing, approve, execute one change, read back and verify, snapshot again, diff against the pre-change snapshot, log. Never batch multiple changes into one API call. If a change has an unexpected effect, you need to know exactly which change caused it.

This loop is not bureaucratic overhead. It is what lets you roll back in minutes instead of hours when something breaks.

### The Change-Control Loop (one iteration per change)

**Step A: Write down the proposed change before touching the gateway**

In `~/unifi-config/assessment-findings.md` (or a separate change record), document:

- What you are changing and why (finding reference)
- The object ID(s) being modified or created
- The exact before-state (copy the relevant JSON from your snapshot)
- The exact after-state (what the object should look like post-change)
- The test you will run to verify the change worked
- The rollback action if it does not work

Get this in writing and reviewed by whoever owns the environment before proceeding. For a solo engagement, at minimum write it down and re-read it yourself before executing.

**Step B: Tag the pre-change snapshot**

You already have a snapshot from Phase 2. Before each new change, note the snapshot directory path in your change log:

```bash
echo "$(date -Iseconds) PRE-CHANGE snapshot=${SNAPSHOT_DIR} change=<description>" \
  >> ~/unifi-config/change-log.txt
```

If significant time has passed since the Phase 2 snapshot or other changes have been made, take a fresh snapshot using the same commands from Phase 2 into a new dated directory.

---

### Steps: Creating a New ZBF Policy (Worked Example)

The following example adds an ALLOW policy that permits traffic from the Internal zone to a specific address group (for example, a server subnet in the DMZ zone). Adapt field values to your actual change.

**Step 1: Create a firewall address group to hold the target IP range (classic API)**

This step creates a named group of IP addresses that the ZBF policy will reference. Creating the group first and then referencing it by ID gives you a reusable object you can update later without editing every policy that uses it.

```bash
curl -sk -X POST \
  "https://${UNIFI_GW}/proxy/network/api/s/default/rest/firewallgroup" \
  -H "X-API-Key: $UNIFI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "grp-dmz-servers",
    "group_type": "address-group",
    "group_members": ["192.0.2.10", "192.0.2.11"]
  }' | python3 -m json.tool | tee /tmp/new-group.json
```

Replace the IP addresses with the actual hosts or subnets you want to target (using your real IP ranges, not the 192.0.2.x documentation addresses shown here). Record the `_id` value from the response:

```bash
GROUP_ID=$(jq -r '.data._id' /tmp/new-group.json)
echo "New group ID: $GROUP_ID"
```

**Step 2: Write the new policy JSON to a file**

Before POSTing, write the payload to a file so you have a record of exactly what you sent:

```bash
cat > /tmp/new-policy.json << EOF
{
  "name": "Allow Internal to DMZ Servers",
  "action": "ALLOW",
  "enabled": true,
  "src_zone": "<SOURCE_ZONE_ID>",
  "dst_zone": "<DESTINATION_ZONE_ID>",
  "src": {
    "matching_target": "ANY",
    "matching_target_type": "SPECIFIC"
  },
  "dst": {
    "matching_target": "IP",
    "matching_target_type": "OBJECT",
    "ip_group_id": "$GROUP_ID",
    "ips": []
  },
  "schedule": {
    "mode": "ALWAYS"
  }
}
EOF
```

Replace `<SOURCE_ZONE_ID>` and `<DESTINATION_ZONE_ID>` with the actual `_id` values from `zbf-zones.json` for the zones you intend to use.

**Critical note on `create_allow_respond`:** For ALLOW policies, do not include `create_allow_respond: false`. For BLOCK policies, you MUST include `"create_allow_respond": false` or the gateway will return error `api.err.FirewallPolicyCreateRespondTrafficPolicyNotAllowed`.

**Step 3: POST the new policy**

```bash
curl -sk -X POST \
  "https://${UNIFI_GW}/proxy/network/v2/api/site/default/firewall-policies" \
  -H "X-API-Key: $UNIFI_API_KEY" \
  -H "Content-Type: application/json" \
  -d @/tmp/new-policy.json | python3 -m json.tool | tee /tmp/new-policy-response.json
```

Record the policy `_id` from the response:

```bash
POLICY_ID=$(jq -r '._id // .data._id' /tmp/new-policy-response.json)
echo "New policy ID: $POLICY_ID"
echo "$(date -Iseconds) CREATED policy_id=${POLICY_ID} name='Allow Internal to DMZ Servers'" \
  >> ~/unifi-config/change-log.txt
```

**Step 4: Read back the policy immediately and verify its position**

A 201 response from the POST means the gateway accepted the object. It does NOT mean the rule is in the right position. In ZBF with first-match evaluation, a new custom ALLOW rule that lands AFTER the zone's default BLOCK has no effect: traffic is dropped before the rule is ever reached.

```bash
# Pull fresh policies and find your new rule's index
curl -sk \
  "https://${UNIFI_GW}/proxy/network/v2/api/site/default/firewall-policies" \
  -H "X-API-Key: $UNIFI_API_KEY" \
  | jq 'if type == "array" then . else .data end
         | to_entries
         | map(select(.value._id == "'$POLICY_ID'"))
         | .[] | {index: .key, id: .value._id, name: .value.name, action: .value.action, enabled: .value.enabled}'
```

If the policy appears after the zone's default BLOCK entry, it is inert. You must reorder it.

**Step 5: Reorder the policy if needed**

Reordering via the API (`PUT /firewall-policies/batch-reorder`) is the official method but is brittle in practice. After two failed attempts via API, stop and use the UI:

1. Open the gateway web UI.
2. Go to Settings > Security > Zone-Based Firewall.
3. Find the policy you created (search by name).
4. Drag it to the correct position (above the default BLOCK for the relevant zone pair).
5. After saving in the UI, re-run step 4 above to verify the new index via API.

**Wait 1 to 2 minutes** after any reorder before testing. The gateway compiles the ruleset and there is a provisioning lag during which the running firewall uses the old compiled ruleset. Testing too early can give you a false negative.

**Step 6: Functional verification**

After confirming the rule is in the correct position, test the traffic the rule is supposed to allow. From a host in the source zone, attempt a connection to the destination:

```bash
# Example: test TCP reachability from a source host
# (run this on a host in the source zone, not on your workstation)
curl -m 5 http://192.0.2.10:80
```

Use your actual destination IP and port. Confirm the connection succeeds.

If the traffic is being blocked when it should be allowed, check:

1. Is the rule enabled? (`"enabled": true` in the read-back)
2. Is the rule above the default BLOCK?
3. Did you wait for provisioning to complete?
4. Is the destination host actually in the destination zone? Cross-check against `zbf-zones.json`.

**Step 7: Post-change snapshot and diff**

Take a new snapshot immediately after verifying the change:

```bash
POST_SNAPSHOT_DIR=~/unifi-config/snapshots/$(date +%Y%m%d-%H%M%S)-post-change
mkdir -p "$POST_SNAPSHOT_DIR"
# Re-run the same curl commands from Phase 2 step 3 and 4, pointing to $POST_SNAPSHOT_DIR
```

Then diff the policy files to confirm only your intended object changed:

```bash
diff \
  <(jq 'if type == "array" then . else .data end | sort_by(._id)' "${SNAPSHOT_DIR}/zbf-policies.json") \
  <(jq 'if type == "array" then . else .data end | sort_by(._id)' "${POST_SNAPSHOT_DIR}/zbf-policies.json")
```

The diff should show only your new policy being added. If you see other changes, investigate before proceeding to the next change.

**Step 8: Log the completed change**

```bash
echo "$(date -Iseconds) VERIFIED policy_id=${POLICY_ID} test=PASS pre_snapshot=${SNAPSHOT_DIR} post_snapshot=${POST_SNAPSHOT_DIR}" \
  >> ~/unifi-config/change-log.txt
```

Repeat this entire loop for each subsequent change.

---

### VLAN Zone Assignment: Special Warning

If any of your changes involve moving a VLAN from one zone to another (for example, moving a VLAN from Internal into a new Management zone), you MUST do the following before making the change:

1. List every host on that VLAN by checking DHCP leases, static assignments, and any monitoring you have. The client list in the UI and the API will show unnamed DHCP entries with only an IP and MAC. Each one of those is a real device. You will not be told which ones are production servers.

2. For each host, identify every inbound flow it depends on (what other hosts initiate connections to it, on what ports). This information is not available from the gateway itself; you need it from whoever operates those systems or from a network flow tool.

3. Moving a VLAN into a default-deny zone silently cuts ALL inbound flows to every host on that VLAN the moment the change is applied. There is no grace period and no warning.

4. After the move, immediately check BLOCK hit counters in the gateway UI (Settings > Security > Zone-Based Firewall, then look at rule hit counts). A sudden rise in a BLOCK counter after a zone move means legitimate traffic is being dropped.

5. Have your rollback ready before you execute (see Phase 5).

A real incident: a VLAN believed to be empty actually held a live server cluster. The zone move silently blocked all inbound connections to that cluster. The issue was caught within minutes because the operator watched BLOCK hit counters immediately after the change and rolled back before users noticed. If the operator had walked away after seeing the 200 response, the outage would have continued until a helpdesk call came in.

### Expected result

After each change-control loop iteration:

- The new or modified object exists in the API read-back.
- The object is in the correct position (correct index for policies).
- Functional testing confirms the intended traffic behavior changed as planned.
- The change log has an entry with timestamp, object ID, test result, and snapshot paths.
- The diff between pre- and post-change snapshots shows only the intended delta.

### If it goes wrong

**POST returns HTTP 422 or an `api.err.*` error:** Read the error body carefully. Common errors:

- `FirewallPolicyCreateRespondTrafficPolicyNotAllowed`: You included or omitted `create_allow_respond` incorrectly. For BLOCK policies, add `"create_allow_respond": false`.
- `InvalidObject`: A field name or value is wrong. Compare your payload shape against an existing policy you pulled in Phase 2 (same API version).
- `zone not found`: The `_id` values for zones in your payload do not match any zone on the gateway. Re-pull `zbf-zones.json` and confirm the IDs.

**Policy was created (201) but traffic is still blocked:** Verify rule order (Step 4 above). The rule is almost certainly landing after the default BLOCK. Reorder and wait 1 to 2 minutes.

**`batch-reorder` returns an error after two attempts:** Stop using the API for reordering. Use the UI drag interface as described in Step 5.

---

## Phase 5: Verification and Rollback

### What this does and why it matters

After completing your planned changes, this phase closes the loop: you do a final verification pass to confirm everything is in the intended state, and you document how to roll back each change if a problem surfaces later. Keeping rollback instructions means any qualified technician can undo a change, not just the person who made it.

### Steps: Final Verification

1. Take a final snapshot using the Phase 2 commands into a new directory named to indicate it is end-of-session:

   ```bash
   FINAL_SNAPSHOT=~/unifi-config/snapshots/$(date +%Y%m%d-%H%M%S)-final
   mkdir -p "$FINAL_SNAPSHOT"
   # Run the full Phase 2 pull commands, pointing to $FINAL_SNAPSHOT
   ```

2. Diff the final snapshot against the pre-change snapshot to produce a complete delta of everything changed in this session:

   ```bash
   for filename in zbf-policies.json zbf-zones.json zbf-zone-matrix.json \
                   firewall-groups.json port-forwards.json networks.json; do
     echo "=== $filename ==="
     diff \
       <(jq 'if type == "array" then . else .data end | sort_by(._id)' \
           "${SNAPSHOT_DIR}/$filename" 2>/dev/null || jq 'sort_by(._id)' "${SNAPSHOT_DIR}/$filename") \
       <(jq 'if type == "array" then . else .data end | sort_by(._id)' \
           "${FINAL_SNAPSHOT}/$filename" 2>/dev/null || jq 'sort_by(._id)' "${FINAL_SNAPSHOT}/$filename") \
       | head -80
     echo ""
   done
   ```

   Review the diff output. Every change you see should correspond to a logged entry in `~/unifi-config/change-log.txt`. If you see changes you did not make, investigate before concluding the session.

3. Run reachability tests for each traffic flow you changed. Organize these as a checklist:

   ```
   [ ] Traffic that should now be ALLOWED: test it, confirm it reaches the destination
   [ ] Traffic that should still be BLOCKED: test it, confirm it is blocked
   [ ] Traffic that was not in scope: test a sample, confirm it was not affected
   ```

4. Check BLOCK hit counters for any unexpectedly rising counts. Log in to the gateway UI, go to Settings > Security > Zone-Based Firewall, and look at the hit count column for BLOCK rules. If a BLOCK rule that had 0 hits before your changes now has hundreds of hits, traffic that used to flow is now being dropped.

5. Confirm your change log is complete:

   ```bash
   cat ~/unifi-config/change-log.txt
   ```

   Every change should have at minimum: timestamp, object ID, action (CREATED / MODIFIED / DELETED), test result, and snapshot references.

6. Archive the snapshot directory:

   ```bash
   tar czf ~/unifi-config/session-$(date +%Y%m%d).tar.gz ~/unifi-config/snapshots/
   echo "Archived snapshots to ~/unifi-config/session-$(date +%Y%m%d).tar.gz"
   ```

   Keep this archive until you are confident the changes have been in production long enough (at least 72 hours) without any issues being reported.

### Expected result

The final diff matches your change log exactly. Reachability tests all pass. No unexpected BLOCK hit counter increases. Change log is written and complete. Snapshot archive is saved.

---

### Steps: Rollback

Roll back only if: functional testing shows a regression, BLOCK hit counters reveal unintended drops, or monitoring alerts indicate an outage linked to your changes. Roll back the most recent change first.

**Rollback option 1: Undo a created policy (delete it)**

If you added a policy and want to remove it:

```bash
# Replace $POLICY_ID with the _id you recorded in the change log
curl -sk -X DELETE \
  "https://${UNIFI_GW}/proxy/network/v2/api/site/default/firewall-policies/${POLICY_ID}" \
  -H "X-API-Key: $UNIFI_API_KEY"
```

Read back to confirm the policy is gone:

```bash
curl -sk \
  "https://${UNIFI_GW}/proxy/network/v2/api/site/default/firewall-policies/${POLICY_ID}" \
  -H "X-API-Key: $UNIFI_API_KEY"
```

A 404 response confirms deletion.

**Rollback option 2: Restore a modified policy to its pre-change state**

Get the original object from the pre-change snapshot:

```bash
# Find the original policy by ID and write it to a temp file
jq 'if type == "array" then . else .data end
    | .[] | select(._id == "'$POLICY_ID'")' \
  "${SNAPSHOT_DIR}/zbf-policies.json" > /tmp/rollback-policy.json

cat /tmp/rollback-policy.json
```

PUT it back:

```bash
curl -sk -X PUT \
  "https://${UNIFI_GW}/proxy/network/v2/api/site/default/firewall-policies/${POLICY_ID}" \
  -H "X-API-Key: $UNIFI_API_KEY" \
  -H "Content-Type: application/json" \
  -d @/tmp/rollback-policy.json | python3 -m json.tool
```

**Rollback option 3: Restore a modified classic-API object (for example, a firewall group)**

```bash
OBJECT_ID="<the _id of the group>"

jq '.data[] | select(._id == "'$OBJECT_ID'")' \
  "${SNAPSHOT_DIR}/firewall-groups.json" > /tmp/rollback-group.json

curl -sk -X PUT \
  "https://${UNIFI_GW}/proxy/network/api/s/default/rest/firewallgroup/${OBJECT_ID}" \
  -H "X-API-Key: $UNIFI_API_KEY" \
  -H "Content-Type: application/json" \
  -d @/tmp/rollback-group.json | python3 -m json.tool
```

**After any rollback:**

1. Wait 1 to 2 minutes for provisioning.
2. Re-run the reachability test that revealed the problem.
3. Check BLOCK hit counters again.
4. Log the rollback:

   ```bash
   echo "$(date -Iseconds) ROLLBACK policy_id=${POLICY_ID} reason='<describe what broke>' result=PASS" \
     >> ~/unifi-config/change-log.txt
   ```

5. Take a fresh snapshot and diff it against the original pre-change snapshot to confirm you are back to baseline.

### If it goes wrong during rollback

**DELETE returns HTTP 404:** The policy ID is wrong, or the policy was already deleted (possibly by someone else in the UI). Confirm by listing all policies and searching for the name.

**PUT returns HTTP 400 or 422:** The object from the snapshot has fields the current firmware version rejects (for example, fields added in a firmware you were not yet on when the snapshot was taken, or fields the v2 API no longer accepts). Try removing any read-only fields such as `_id`, `site_id`, or `create_time` from the payload before re-sending. If still failing, recreate the object manually from the known-good values in the snapshot.

**Traffic is still broken after rollback:** Confirm you waited 1 to 2 minutes for provisioning. If still broken, check whether another rule is shadowing the one you restored (a BLOCK rule in an earlier position that was not part of your changes). Use the API to pull the current policy list and inspect the index order around the restored rule.

---

## Appendix A: Shell Helpers

These functions are optional. Add them to `~/unifi-config/helpers.sh` and source it in your session.

```bash
#!/usr/bin/env bash
# Source this file: source ~/unifi-config/helpers.sh
# Requires UNIFI_GW and UNIFI_API_KEY to be set in environment

source ~/unifi-config/.env

V2="https://${UNIFI_GW}/proxy/network/v2/api/site/default"
CL="https://${UNIFI_GW}/proxy/network/api/s/default"

# Quick connectivity probe
unifi_ping() {
  curl -sk "${V2%/v2*}/integration/v1/sites" \
    -H "X-API-Key: $UNIFI_API_KEY" | python3 -m json.tool
}

# Pull a single endpoint and pretty-print
unifi_get() {
  local path="$1"  # e.g. /firewall-policies or /rest/firewallgroup
  local base
  if [[ "$path" == /rest/* ]] || [[ "$path" == /stat/* ]] || [[ "$path" == /get/* ]]; then
    base="$CL"
  else
    base="$V2"
  fi
  curl -sk "${base}${path}" -H "X-API-Key: $UNIFI_API_KEY" | python3 -m json.tool
}

# Get policy by ID
unifi_policy() {
  curl -sk "${V2}/firewall-policies/$1" \
    -H "X-API-Key: $UNIFI_API_KEY" | python3 -m json.tool
}

# Delete policy by ID
unifi_delete_policy() {
  curl -sk -X DELETE "${V2}/firewall-policies/$1" \
    -H "X-API-Key: $UNIFI_API_KEY"
  echo "Deleted (or not found): $1"
}
```

---

## Appendix B: Common jq Recipes for JSON Analysis

```bash
# Extract all custom (non-predefined) ALLOW policies with their zone IDs
jq 'if type == "array" then . else .data end
    | [.[] | select(.predefined != true and .action == "ALLOW")]
    | .[] | {name, id: ._id, src_zone, dst_zone, enabled}' \
  "${SNAPSHOT_DIR}/zbf-policies.json"

# Find policies where a specific zone is the source
ZONE_ID="<your zone _id>"
jq --arg z "$ZONE_ID" \
  'if type == "array" then . else .data end
   | [.[] | select(.src_zone == $z)]
   | .[] | {name, action, dst_zone, enabled}' \
  "${SNAPSHOT_DIR}/zbf-policies.json"

# List all VLANs with their subnet
jq '.data[] | select(.ip_subnet != null) | {name, vlan: .vlan, subnet: .ip_subnet, purpose}' \
  "${SNAPSHOT_DIR}/networks.json"

# Show zone matrix as a readable table
jq 'to_entries | .[] | {from: .key, actions: (.value | to_entries | .[] | {to: .key, action: .value})}' \
  "${SNAPSHOT_DIR}/zbf-zone-matrix.json"
```

---

## Appendix C: Quick-Reference Checklist

Use this checklist as a pre-flight and post-flight checklist for each session.

**Before starting any work:**

- [ ] Connectivity probe passes (Phase 1, Step 7)
- [ ] Snapshot taken and all files verified non-empty (Phase 2)
- [ ] Findings documented in writing (Phase 3)
- [ ] Change list reviewed and approved

**Before each change:**

- [ ] Pre-change snapshot exists (or is fresh, under 2 hours old)
- [ ] Change proposal is written down with before/after state and rollback plan
- [ ] You know which zone(s) and hosts are affected
- [ ] For VLAN zone moves: every host on the VLAN has been identified and all inbound flows documented

**After each change:**

- [ ] Policy/object was created (201 / 200 response)
- [ ] Rule is in the correct position (not shadowed by a BLOCK above it)
- [ ] Waited 1 to 2 minutes for provisioning
- [ ] Functional test passed
- [ ] BLOCK hit counters checked for unexpected increases
- [ ] Post-change snapshot taken
- [ ] Diff reviewed: only intended changes appear
- [ ] Change log entry written

**End of session:**

- [ ] Final snapshot taken
- [ ] Session diff reviewed
- [ ] Change log complete
- [ ] Snapshot archive saved
- [ ] Rollback procedures written for each change

---

*End of runbook.*
