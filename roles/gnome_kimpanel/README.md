# gnome_kimpanel

GNOME Shell 拡張機能「Input Method Panel」(kimpanel,
UUID `kimpanel@kde.org`) をデスクトップユーザにインストールし、有効化する
ロール。

kimpanel は fcitx5 / IBus の変換候補ウィンドウと状態表示を GNOME Shell の
ネイティブパネルとして描画する。Wayland では fcitx5 が自前の classic UI を
安定して描画できないため、これが事実上の必須コンポーネントになる。

fcitx5 は KDE kimpanel D-Bus プロトコルを標準で話せる (fcitx5 の `dbus`
アドオンに内蔵) ので、fcitx5 側に追加パッケージは不要。本ロールは GNOME 側の
拡張機能のみを扱う。`fcitx5_mozc` ロールと組み合わせて使う想定。

## 動作

- 稼働中の GNOME Shell のメジャーバージョンを検出し、
  extensions.gnome.org の `extension-info` API から対応ビルドの zip を取得する
  (バージョンタグを手で固定しないので GNOME のアップグレードに追従する)。
- `gnome-extensions install --force` でユーザの
  `~/.local/share/gnome-shell/extensions/` に展開する。
- `org.gnome.shell enabled-extensions` に UUID を追記して有効化する。

> **Wayland 注意**: 新規インストールした拡張は稼働中の Shell が再スキャン
> するまで認識されないため `gnome-extensions enable` は失敗する。本ロールは
> gsettings のリストへ直接追記するため、**次回ログインで有効になる**。
> 適用後はログアウト/ログインが必要。

## 主な変数 (`defaults/main.yml`)

| 変数 | 既定 | 説明 |
| --- | --- | --- |
| `gnome_kimpanel_uuid` | `kimpanel@kde.org` | 拡張の UUID |
| `gnome_kimpanel_ego_base` | `https://extensions.gnome.org` | EGO のベース URL |
| `gnome_kimpanel_enable` | `true` | enabled-extensions へ追記して有効化 |
| `gnome_kimpanel_upgrade` | `true` | EGO 側の version が異なれば再インストール |

playbook 側で以下を渡す必要がある (fcitx5_mozc と同じパターン):

- `gnome_kimpanel_user` … デスクトップユーザ名 (通常 SUDO_USER)
- `gnome_kimpanel_user_home` … そのユーザのホーム

## 使い方

`ubu26_desktop.yml` に組み込み済み。単体で流す場合:

```bash
ansible-playbook -i hosts playbooks/conf/linux/ubu26_desktop.yml \
  -e server=localhost --connection=local -K --tags gnome_kimpanel
```
