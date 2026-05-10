pdf_tools
=========

Debian/Ubuntu 系で日常的な PDF 操作 (閲覧/注釈、結合・分割・回転、CLI 処理)
を一発で揃える apt ベースのバンドルロール。Adobe Acrobat (Web 版) との
補完を意識し、OCR / 高品質 Office 変換 / 電子署名のような重い処理は Adobe
側に任せる前提でローカルツールに絞っている。

含まれるもの
------------

| カテゴリ | パッケージ | 役割 |
| --- | --- | --- |
| 閲覧 | `okular` | KDE 製 PDF ビューワ。ハイライト/コメント/付箋/手書き、PDF フォーム入力対応 |
| GUI 編集 | `pdfarranger` | GTK でドラッグ&ドロップ結合・抽出・回転・並び替え |
| CLI | `qpdf` | モダンな結合/分割/暗号化/最適化 |
| CLI | `poppler-utils` | `pdftotext`, `pdftoppm`, `pdfinfo`, `pdfimages` 等 |
| CLI | `ghostscript` | 圧縮・修復・PostScript 変換 |

各カテゴリは変数で個別に無効化可能。

Requirements
------------

- Ansible 2.9 以上
- Debian / Ubuntu 系 (apt)
- `become: yes`

Role Variables
--------------

| 変数 | デフォルト | 説明 |
| --- | --- | --- |
| `pdf_tools_install_viewer` | `true` | 閲覧ツール (Okular) を入れるか |
| `pdf_tools_viewer_packages` | `[okular]` | 閲覧パッケージ群 |
| `pdf_tools_install_editor` | `true` | GUI 編集ツール (PDF Arranger) を入れるか |
| `pdf_tools_editor_packages` | `[pdfarranger]` | GUI 編集パッケージ群 |
| `pdf_tools_install_cli` | `true` | CLI ツール (qpdf 他) を入れるか |
| `pdf_tools_cli_packages` | `[qpdf, poppler-utils, ghostscript]` | CLI パッケージ群 |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - pdf_tools
```

CLI のみ (サーバや WSL 等) で十分な場合:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: pdf_tools
      vars:
        pdf_tools_install_viewer: false
        pdf_tools_install_editor: false
```

便利な定型例
------------

```bash
# 結合
qpdf --empty --pages a.pdf b.pdf c.pdf -- merged.pdf

# ページ抽出 (1-3 ページだけ)
qpdf in.pdf --pages . 1-3 -- out.pdf

# テキスト抜き出し
pdftotext in.pdf -

# 軽量化 (Ghostscript)
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dPDFSETTINGS=/ebook \
   -dNOPAUSE -dQUIET -dBATCH -sOutputFile=out.pdf in.pdf

# サムネイル PNG 生成
pdftoppm -png -r 150 in.pdf thumb
```

Tags
----

- `pdf_tools`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
