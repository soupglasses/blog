---
title: Mikrotik DHCP to DNS synchronization script
date: 2025-08-09
authors:
  - name: SoupGlasses
    link: https://finnes.dev
    image: https://github.com/soupglasses.png
---
For some reason i fail to understand, the DHCP server shipped with Mikrotik still has no ability to sync its active hostname leases dynamically to the DNS server itself. So this quite basic LAN feature is something every Mikrotik administrator will have to implement themselves using the Mikrotik scripting tools.

How this script functions to sync its DHCP to DNS hostnames is heavily on the work from Daniel Aleksandersen at <https://www.ctrl.blog/entry/routeros-dhcp-lease-script.html>. I recommend reading it for a explanation for why the script is written in this more way.

The TL;DR is, due to how many bad DHCP clients exist in the world, all DHCP triggered scripts fail to maintain a DNS to DHCP association accurate over time. So instead the idea is to focus on creating a system level script that walks all the DHCP leases at a regular interval, then cleaning up the hanging leases after the walk.

You can still trigger the script automatically from the DHCP lease side using a light wrapper, which you can find below the main script.

## System script `dhcp-to-dns`

```bash {{filename="dhcp-to-dns"}}
# SPDX-License-Identifier: CC0-1.0
# DHCP to DNS synchronization script
# Handles multiple host addresses using MAC address tracking
# Author: SoupGlasses
# Version: 2.2.0

:local domains [:toarray "home.arpa"]
:local dnsttl "10m"
:local magiccommentbase "from-dhcp"

# Track active MAC addresses for the cleanup process.
:local activemacs [:toarray ""]

# Process phase:
:foreach lease in [/ip dhcp-server lease find] do={
  :local hostname [/ip dhcp-server lease get value-name=host-name $lease]
  :local hostaddr [/ip dhcp-server lease get value-name=address $lease]
  :local hostmac [/ip dhcp-server lease get value-name=mac-address $lease]

  # Only process leases with a valid hostname and MAC.
  :if ([:len $hostname] > 0 && [:len $hostmac] > 0) do={
    :local magiccomment "$magiccommentbase ($hostmac)"
    :set activemacs ($activemacs, $hostmac)

    :foreach domain in $domains do={
      :local fqdn "$hostname.$domain"

      :local existingentry [/ip dns static find where name=$fqdn comment=$magiccomment]

      :if ([:len $existingentry] = 0) do={
        :do {
          :log info "DNS: Adding $fqdn -> $hostaddr ($hostmac)"
          /ip dns static add name=$fqdn address=$hostaddr comment=$magiccomment ttl=$dnsttl
        } on-error={
          :log warning "DNS: Failed to add $fqdn -> $hostaddr ($hostmac) - entry already exists"
        }
      } else={
        :local currentaddr ""
        :do {
          :set currentaddr [/ip dns static get value-name=address [:pick $existingentry 0]]
        } on-error={
          :set currentaddr ""
        }

        :if ($currentaddr != $hostaddr) do={
          :log info "DNS: Updating $fqdn: $currentaddr -> $hostaddr ($hostmac)"
          /ip dns static set address=$hostaddr [:pick $existingentry 0]
        } else={
          # Entry is already correct, simply continue.
        }
      }
    }
  }
}

# Cleanup phase:
:foreach dnsentry in [/ip dns static find where comment~"^$magiccommentbase \\("] do={
  :local entrycomment [/ip dns static get value-name=comment $dnsentry]
  :local entryname [/ip dns static get value-name=name $dnsentry]

  :local macstart ([:find $entrycomment "("] + 1)
  :local macend [:find $entrycomment ")" $macstart]

  :if ($macstart > 0 && $macend > $macstart) do={
    :local extractedmac [:pick $entrycomment $macstart $macend]

    :if ([:type [:find $activemacs $extractedmac]] = "nil") do={
      :log info "DNS: Removing stale entry $entryname ($extractedmac)"
      /ip dns static remove $dnsentry
    }
  } else={
    :log warning "DNS: Found entry with malformed comment: $entryname - $entrycomment"
  }
}
```

## DHCP script

Handy if you are an impatient admin like me, and you run your sync script at a slower interval acting more like a clean-up process of badly acting devices. Only really sensible to use in small home environments where DHCP lease updates happen infrequently.

```rsc
:if ($leaseBound = 0 || $leaseBound = 1) do={
    /system script run dhcp-to-dns
}
```

## Improvements for the future

- Use `Domain` from DHCP Networks directly, instead of using a `domains` array. I do not use multiple local domains.
- DHCP Active host name validation and normalization. Right now it just YOLOs it direct into DNS, errors be errors. Works since my local network is too small to see the pains of my shortcuts.
