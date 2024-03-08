# Instructions

> [!NOTE]
> This PKGBUILD is a fork of [ariel-miculas/cisco-secureclient](https://github.com/ariel-miculas/cisco-secureclient) to support installation from organization's self hosted installer script.

1. Download the latest secureclient installer script (`cisco-secure-client-linux64-${pkgver}-core-vpn-webdeploy-k9.sh`) from your organization's page
2. Copy the script to `$HOME/work/cisco-anyconnect/` or change the `source` variable in PKGBUILD to point to the location of your downloaded script
3. You may have to remove `/opt/cisco/secureclient/lib` if there was a previous failed upgrade of secureclient
4. Run `makepkg -si` (or any other AUR helper's equivalent) to install the package
5. Check the status of the vpn service with `systemctl status vpnagentd.service`. You may have to start it using `systemctl start vpnagentd.service` or permanently enable it using `systemctl enable --now vpnagentd.service`
6. Run `cisco-secureclient&` or start it using the GUI
