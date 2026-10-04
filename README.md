# Solaris 10 SPARC on QEMU sun4u

QEMU上のSPARC64環境においてSolaris 10を動かすためのQEMU 11.1.1とOpenBIOS用のパッチです。


## ビルド

必要なコンパイラとビルドツールを導入して、実行環境を整えてください。

OpenBIOS（Linux環境。WindowsではWSLを使用。SPARC64クロスコンパイラ、`xsltproc`、`fcode-utils`が必要）:

```sh
git clone https://github.com/openbios/openbios.git
cd openbios
git checkout --detach e5ac46dd24e6216c36aa80462af25457e7029440
git apply --check ../openbios-sun4u-solaris10.patch
git apply ../openbios-sun4u-solaris10.patch
./config/scripts/switch-arch sparc64
make
cp obj-sparc64/openbios-builtin.elf ../openbios.elf
cd ..
```

QEMU（WindowsではMSYS2 UCRT64を使用）:

ビルド前にWindowsの開発者モードを有効にしてください。

```sh
git clone --branch v11.1.1 --depth 1 https://gitlab.com/qemu-project/qemu.git qemu-11.1.1
cd qemu-11.1.1
git apply --check ../qemu-11.1.1-solaris10-sun4u.patch
git apply ../qemu-11.1.1-solaris10-sun4u.patch
mkdir build
cd build
../configure --target-list=sparc64-softmmu
ninja
cd ../..
```

## インストール/起動
インストールにはSolaris 10 SPARC版DVDのISOイメージが必要になります。各自の責任のもとに用意してください。

生成した`openbios.elf`とDVDのISOイメージを、以下のコマンドを実行するディレクトリに置いてください。WSLでビルドした場合は、`openbios.elf`をWindows側へコピーします。

(例)
```sh
./qemu-11.1.1/build/qemu-img.exe create -f qcow2 solaris10.qcow2 40G
./qemu-11.1.1/build/qemu-system-sparc64.exe \
  -M sun4u -m 3048 \
  -drive file=solaris10.qcow2,format=qcow2,if=ide,bus=0,unit=0,media=disk \
  -drive file=sol-10-u11-ga-sparc-dvd.iso,format=raw,if=ide,bus=1,unit=0,media=cdrom,readonly=on \
  -boot d -nographic -vga none -bios openbios.elf \
  -prom-env 'input-device=ttya' -prom-env 'output-device=ttya' \
  -nic none
```

インストール後は`-boot d`を`-boot c`に変更してください。ネットワークを使う場合は`-nic none`を環境に合うネットワーク設定に置き換えてください。

## 動作確認内容

Solaris 10のSPARC64環境へのインストール、UFS/ZFSからの起動、シリアル入力、ネットワーク、リモート接続(telnet,デスクトップ)の動作確認をしています。

## 判明している制限

フロッピーは未サポートです。
`init 0`後のPROMからの再ブートも未対応のため、QEMUを終了して起動し直してください。
グラフィックス

## ライセンス

対象ソースのライセンスに従います。GPL v2の文面は`COPYING`を参照してください。

## 商標

Solarisは，Oracle Corporation 及びその子会社，関連会社の米国 及びその他の国における登録商標または商標です。
本プロジェクトはOracleとは関係のない個人プロジェクトです。
