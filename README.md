<p align="center">
  <a href="https://github.com/lupaxa-security-toolbox">
    <img src="https://raw.githubusercontent.com/the-lupaxa-project/brand-assets/master/logos/organisations/security-toolbox/readme-logo.png" alt="Security Toolbox" />
  </a>
</p>

<h1 align="center">Tcp Wrapper Multiplexer</h1>

Run several TCP Wrapper filter scripts in order, and return the first deny.

This wrapper does not decide allow or deny itself. It calls each configured filter with the client address.

A filter exit code of `0` means allow, and the multiplexer continues. Any other exit code means deny, and that result is returned immediately. If every filter allows, the multiplexer allows.

> [!NOTE]
> TCP Wrappers do not replace a firewall. Use this script as one layer of a larger control.

## Install

Copy `src/multiplexer.sh` to `/usr/local/sbin/multiplexer` and make it executable:

```bash
sudo install -m 755 src/multiplexer.sh /usr/local/sbin/multiplexer
```

With `FILTERS` empty, the script allows the connection. Nothing is denied until you name at least one filter.

## Adding Filters

Set `FILTERS` in `src/multiplexer.sh` to a space-separated or comma-separated list of filter executable names. Each name is run from `FILTER_PATH` (default `/usr/local/sbin`).

```bash
FILTERS="asn-filter country-filter"
FILTER_PATH="/usr/local/sbin"
```

Filters run in the order listed. Each filter is called as:

```bash
/usr/local/sbin/<filter> <client-ip> MUX
```

A filter that is missing or not executable is skipped.

## TCP Wrapper Order

TCP Wrappers read `/etc/hosts.allow` first, then `/etc/hosts.deny`. Anything not handled in `hosts.allow` falls through to `hosts.deny`.

### Hosts Allow

Pass every SSH client address to the multiplexer. `aclexec` runs the script, and `%a` is the client address. Exit `0` allows the connection. Exit `1` denies it.

```text
sshd: ALL: aclexec /usr/local/sbin/multiplexer %a
```

### Hosts Deny

Deny SSH when `hosts.allow` does not allow it:

```text
sshd: ALL
```

> [!NOTE]
> The deny rule should not be reached when the multiplexer handles every address. Keep it as a fallback.

## Development

```bash
make init
make bash-check
```

<a href="https://github.com/the-lupaxa-project">
    <img src="https://raw.githubusercontent.com/the-lupaxa-project/brand-assets/master/logos/components/footer-for-child-orgs.svg" alt="The Lupaxa Project Footer" width="100%" />
</a>
