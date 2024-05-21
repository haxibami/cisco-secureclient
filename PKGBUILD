# Original Maintainer: K900 <me@0upti.me>
# Original Contributor: Michael Duncan Hammond <mhammond9@radford.edu>
# Maintainer: haxibami <contact at haxibami dot net>

pkgname=cisco-secureclient
pkgver=5.1.3.62
pkgrel=1
pkgdesc='Cisco AnyConnect Secure Mobility Client'
url='https://www.cisco.com/c/en/us/products/security/anyconnect-secure-mobility-client/index.html'
arch=('x86_64')
depends=('libxml2' 'ca-certificates')
optdepends=(
  'hicolor-icon-theme: required by desktop shortcuts'
  'gtk2: required by vpnui'
  'glib2: required by vpnui'
  'gtk3: required by acwebhelper'
  'webkit2gtk: required by acwebhelper'
)
license=('custom')

# stripping the binaries trips some sort of an integrity check inside vpnagentd
options=('!strip')

# you will have to obtain the installer yourself - it's not available publicly
source=(
  "installer.sh::file:///$HOME/work/cisco-anyconnect/cisco-secure-client-linux64-${pkgver}-core-vpn-webdeploy-k9.sh"
  "${pkgname}.sh"
  "AnyConnectLocalPolicy.xml"
)

sha256sums=('b98baaca1b22a5ab09d28af6ae8a6284f85ffb98f6268d443e6fd00aa890e8df'
            'd29746d526ae87d011a1b988531d72a1b0da018872f22b3dd8ebd2fc2082bda2'
            'f356c6bca23bb187e8e6c2d6504d9a0c30f7d45f6c78bef399e67a5d5079343d')

prepare() {
  cd "${srcdir}"

  # extract the core tarball from the installer
  MARKER=$((`grep -an "[B]EGIN\ ARCHIVE" ./installer.sh | cut -d ":" -f 1` + 1))
  MARKER_END=$((`grep -an "[E]ND\ ARCHIVE" ./installer.sh | cut -d ":" -f 1` - 1))
  head -n ${MARKER_END} ./installer.sh | tail -n +${MARKER} | head -c -1 > ./core.tar.gz
  tar -xzf ./core.tar.gz
  rm ./core.tar.gz
}

package() {
  cd "${srcdir}/vpn"

  # install binaries
  for binary in "vpnagentd" "vpn" "vpndownloader" "vpndownloader-cli" "manifesttool_vpn" "acinstallhelper" "vpnui" "acwebhelper" "load_tun.sh"; do
      install -Dm755 ${binary} "${pkgdir}/opt/cisco/secureclient/bin/${binary}"
  done

  # install libs
  for lib in "libvpnagentutilities.so" "libvpncommon.so" "libvpncommoncrypt.so" \
      "libvpnapi.so" "libacruntime.so" "libacciscossl.so" "libacciscocrypto.so" \
      "cfom.so" "libboost_date_time.so" "libboost_filesystem.so" "libboost_regex.so" "libboost_system.so" \
      "libboost_thread.so" "libboost_atomic.so" "libboost_chrono.so" \
      "libaccurl.so.4.8.0"; do
      install -Dm755 ${lib} "${pkgdir}/opt/cisco/secureclient/lib/${lib}"
  done
#   rm -rf ${pkgdir}/opt/cisco/secureclient/lib/libboost*

  # the installer copies all the other symlinks, but creates this one
  # for some reason so let's just create it ourselves
  ln -s /opt/cisco/secureclient/lib/libaccurl.so.4.8.0 "${pkgdir}/opt/cisco/secureclient/lib/libaccurl.so.4"

  # install plugins
  # we intentionally don't install the telemetry plugin here
  # because it tries to write to /opt and we don't want that
  for plugin in "libacwebhelper.so" "libvpnipsec.so"; do
      install -Dm755 ${plugin} "${pkgdir}/opt/cisco/secureclient/bin/plugins/${plugin}"
  done

  cp -R resources "${pkgdir}/opt/cisco/secureclient/resources"

  # install some misc stuff
  install -Dm444 AnyConnectProfile.xsd "${pkgdir}/opt/cisco/secureclient/profile/AnyConnectProfile.xsd"

  for file in "ACManifestVPN.xml" "update.txt" "AnyConnectLocalPolicy.xsd"; do
      install -Dm444 ${file} "${pkgdir}/opt/cisco/secureclient/${file}"
  done

  # install desktop file for vpnui
  install -Dm644 resources/vpnui48.png "${pkgdir}/usr/share/icons/hicolor/48x48/apps/cisco-secure-client.png"
  install -Dm644 resources/vpnui128.png "${pkgdir}/usr/share/icons/hicolor/128x128/apps/cisco-secure-client.png"

  sed -i "s|^Exec=.*|Exec=${pkgname}|g" com.cisco.secureclient.gui.desktop
  install -Dm644 com.cisco.secureclient.gui.desktop "${pkgdir}/usr/share/applications/com.cisco.secureclient.gui.desktop"

  # install license
  for license in "license.txt" "OpenSource.html"; do
      install -Dm644 ${license} "${pkgdir}/usr/share/licenses/${pkgname}/${license}"
  done

  # install systemd unit for vpnagentd
  install -Dm644 "vpnagentd.service" "${pkgdir}/usr/lib/systemd/system/vpnagentd.service"

  install -Dm755 ${srcdir}/${pkgname}.sh ${pkgdir}/usr/bin/${pkgname}
  # install CA certificates
  mkdir -p "${pkgdir}/opt/.cisco/certificates/ca"

  # first, install our own system root
  ln -s /etc/ca-certificates/extracted/tls-ca-bundle.pem "${pkgdir}/opt/.cisco/certificates/ca/system-ca.pem"

  # then, install Cisco's, because it doesn't actually trace to any of the trusted roots we have
  # (thanks, VeriSign)
  install -Dm644 VeriSignClass3PublicPrimaryCertificationAuthority-G5.pem "${pkgdir}/opt/.cisco/certificates/ca/VeriSignClass3PublicPrimaryCertificationAuthority-G5.pem"

  # install custom policy to disable auto updates
  # AnyConnect will attempt to update itself as root, and then run all over both itself and our packaging
  # so prevent it from doing anything like that
  #
  # this may break some really quirky setups that require downloading files from the server,
  # but there's no other way around it that I could find
  install -Dm644 "${srcdir}/AnyConnectLocalPolicy.xml" "${pkgdir}/opt/cisco/secureclient/AnyConnectLocalPolicy.xml"
}
