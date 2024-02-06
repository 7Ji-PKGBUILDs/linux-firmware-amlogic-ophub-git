# Maintainer: 7Ji <pugokughin@gmail.com>

pkgname=linux-firmware-amlogic-ophub-git
pkgver=20231018.db9e26b.20240115.9b6d0b08
pkgrel=1
pkgdesc="Firmware files for Linux - mainly for AArch64 Amlogic platform, complete set, collected by ophub"
arch=('any')
makedepends=('git')
url="https://github.com/ophub/firmware"
license=('GPL2' 'GPL3' 'custom')
conflicts=('linux-firmware-amlogic-ophub')
replaces=('linux-firmware-amlogic-ophub')
options=(!strip)
source=("git+${url}.git#branch=main")
sha256sums=('SKIP')
depends=('linux-firmware')

pkgver() {
  cd firmware

  local _ver_linux_firmware=$(LANG=C pacman -Qi linux-firmware | grep -E '^Version' | awk '{print $3}')
  _ver_linux_firmware="${_ver_linux_firmware##*:}"
  _ver_linux_firmware="${_ver_linux_firmware%%-*}"

  # Commit date + short rev
  printf '%s.%s.%s' \
    $(TZ=UTC git show -s --pretty=%cd --date=format-local:%Y%m%d HEAD) \
    $(git rev-parse --short HEAD) \
    "${_ver_linux_firmware}"
}

build() {
  cd firmware/firmware/brcm
  # gtking/gtking pro is bcm4356 wifi/bluetooth, wifi5 module AP6356S
  sed -e "s/macaddr=.*/macaddr=${random_macaddr}:00/" "brcmfmac4356-sdio.txt" >"brcmfmac4356-sdio.azw,gtking.txt"
  # gtking/gtking pro is bcm4356 wifi/bluetooth, wifi6 module AP6275S
  sed -e "s/macaddr=.*/macaddr=${random_macaddr}:01/" "brcmfmac4375-sdio.txt" >"brcmfmac4375-sdio.azw,gtking.txt"
  # Phicomm N1 is bcm43455 wifi/bluetooth
  sed -e "s/macaddr=.*/macaddr=${random_macaddr}:02/" "brcmfmac43455-sdio.txt" >"brcmfmac43455-sdio.phicomm,n1.txt"
  # MXQ Pro+ is AP6330(bcm4330) wifi/bluetooth
  sed -e "s/macaddr=.*/macaddr=${random_macaddr}:03/" "brcmfmac4330-sdio.txt" >"brcmfmac4330-sdio.crocon,mxq-pro-plus.txt"
  # HK1 Box & H96 Max X3 is bcm54339 wifi/bluetooth
  sed -e "s/macaddr=.*/macaddr=${random_macaddr}:04/" "brcmfmac4339-sdio.ZP.txt" >"brcmfmac4339-sdio.amlogic,sm1.txt"
  # old ugoos x3 is bcm43455 wifi/bluetooth
  sed -e "s/macaddr=.*/macaddr=${random_macaddr}:05/" "brcmfmac43455-sdio.txt" >"brcmfmac43455-sdio.amlogic,sm1.txt"
  # new ugoos x3 is brm43456
  sed -e "s/macaddr=.*/macaddr=${random_macaddr}:06/" "brcmfmac43456-sdio.txt" >"brcmfmac43456-sdio.amlogic,sm1.txt"
  # x96max plus v5.1 (ip1001m phy) adopts am7256 (brcm4354)
  sed -e "s/macaddr=.*/macaddr=${random_macaddr}:07/" "brcmfmac4354-sdio.txt" >"brcmfmac4354-sdio.amlogic,sm1.txt"
}

package() {
  local _prefix="${pkgdir}"/usr/lib
  install -d "${_prefix}"
  cd firmware
  pacman -Ql linux-firmware | sed -n 's|^linux-firmware /usr/lib/\(firmware/.*\)|\1|p' | sort | uniq > linux-firmware.list
  local _file _target _link
  if [[ "${CARCH}" == 'aarch64' ]]; then
    local _nocomp='yes'
  else
    local _nocomp=''
  fi
  for _file in $(find firmware); do
    grep -q "^${_file}\(.zst\|/\)\?$" linux-firmware.list && continue
    if [[ -L "${_file}" ]]; then
      echo "L: ${_file}"
      _target=$(readlink "${_file}")
      if [[ -z "${_nocomp}" && -f "${_file}" ]]; then
        _target+='.zst'
        _file+='.zst'
      fi
      _link="${_prefix}/${_file}"
      install -d "${_link%/*}"
      ln -s "${_target}" "${_link}"
    elif [[ -f "${_file}" ]]; then
      echo "F: ${_file}"
      if [[ "${_nocomp}" ]]; then
        install -Dm644 "${_file}" "${_prefix}/${_file}"
      else
        zstd --compress --quiet --stdout "${_file}" |
          install -Dm644 /dev/stdin "${_prefix}/${_file}.zst"
      fi
    else
      echo "D: ${_file}"
      install -d "${_prefix}/${_file}"
    fi
  done
}
