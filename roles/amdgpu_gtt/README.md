amdgpu_gtt
==========

AMD iGPU の GTT (Graphics Translation Table) サイズを GRUB の
kernel cmdline (`amdgpu.gttsize=N`) で拡張するロール。

iGPU は専用 VRAM を持たず、システム RAM から借りる量 (GTT) が
既定ではモデル全体を載せきれない。14B クラスの GGUF を
Vulkan オフロード (`-ngl 99`) で動かすには 20 GiB 前後の GTT が必要。

**反映には再起動が必須**。このロールは grub.cfg の再生成までで終わる。

Requirements
------------

- Ansible 2.9 以上
- Debian/Ubuntu (`update-grub` 前提)
- 物理機 (VM/コンテナでは効果なし)

Role Variables
--------------

| 変数 | 既定値 | 説明 |
| --- | --- | --- |
| `amdgpu_gtt_enable` | `false` | このロールを実行するかのスイッチ。`true` にしないと何もしない |
| `amdgpu_gtt_size_mb` | `20480` | GTT サイズ (MiB)。14B Q4_K_M を 32GB RAM ホストで動かす想定 |
| `amdgpu_gtt_grub_cmdline` | `quiet splash amdgpu.gttsize={{ amdgpu_gtt_size_mb }}` | `GRUB_CMDLINE_LINUX_DEFAULT` 全体に書き込む文字列。他オプションを保持したい場合は丸ごと上書きすること |

Dependencies
------------

なし。

Example Playbook
----------------

```yaml
- hosts: k16
  become: true
  roles:
    - { role: amdgpu_gtt, amdgpu_gtt_enable: true, amdgpu_gtt_size_mb: 20480 }
```

実行後に `sudo reboot` を忘れずに。再起動後に
`cat /proc/cmdline` で `amdgpu.gttsize=20480` が見えていれば反映済み。

復旧
----

万一起動しなくなった場合は GRUB メニューから旧設定を選ぶか、
別メディアで起動して `/etc/default/grub.<日付>~` を `grub` に戻して
`sudo update-grub` を実行する (`backup: true` を有効にしてあるため
1 世代前は必ず残る)。

License
-------

BSD-3-Clause

Author Information
------------------

dobachi
