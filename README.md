# Ubuntu + WireGuard + RDP (iPhone接続用) 手順

## 前提
- Ubuntu環境
- WireGuard, xrdp を使用
- iPhone で WireGuard と Windos App Mobileアプリ使用


# 1. /etc/wireguard に移動
sudo apt install -y wireguard-tools qrencode
cd /etc/wireguard/

# 2. WireGuard 鍵作成
wg genkey | tee server_privatekey | wg pubkey > server_publickey
wg genkey | tee client_privatekey | wg pubkey > client_publickey

# 3. サーバー設定ファイル wg0.conf
cat << 'EOF' > wg0.conf
[Interface]
PrivateKey=<ここに server privatekey の内容を貼る>
Address=10.0.0.1/24
ListenPort=51820

[Peer]
PublicKey=<ここに client public key>
AllowedIPs=10.0.0.2/32
PersistentKeepalive=25
EOF

# 4. クライアント設定ファイル client.conf
cat << 'EOF' > client.conf
[Interface]
PrivateKey=<ここに client private key>
Address=10.0.0.2/24
DNS=1.1.1.1

[Peer]
PublicKey=<ここに server public key>
Endpoint=<サーバー公開IPまたはVPN経由IP>:51820
AllowedIPs=0.0.0.0/0
PersistentKeepalive=25
EOF

# 5. QRコード生成（表示されたQRをWireGuardアプリで読み込み）
qrencode -t png -o client.png < client.conf

# 6. WireGuard 起動｛｝
sudo wg-quick up wg0

# 7. xrdp インストールと有効化
sudo apt install -y xrdp
sudo systemctl enable xrdp
sudo systemctl start xrdp

# 8. デスクトップ環境 (XFCE) 設定
sudo apt install -y xfce4 xfce4-goodies
echo "startxfce4" > ~/.xsession

### startwm.sh の編集
sudo tee /etc/xrdp/startwm.sh > /dev/null << 'EOF'
#!/bin/sh
. /etc/profile
if test -r ~/.profile; then
        . ~/.profile
fi
exec startxfce4
EOF

# 9. Windows App Mobile に以下を設定後、接続
IP:10.0.0.1
ユーザー名:{PCユーザー名}
パスワード:{PCパスワード}

+ RDP 接続で音声トラブルがある場合
sudo pkill -f ':10 -auth .Xauthority'
sudo pkill -f xrdp-chansrv
sudo pkill -f pw-cli
