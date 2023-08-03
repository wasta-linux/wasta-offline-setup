# Notes about USB flatpak repos

## Documentation
- [Blog announcement](https://blogs.gnome.org/mclasen/2018/08/26/about-flatpak-installations/)
- [Reference docs](https://docs.flatpak.org/en/latest/usb-drives.html)

## Useful Commands
- `flatpak create-usb`: create repo and add flatpak in passed folder
- `flatpak remote-ls "file///path/to/.ostree/repo"`: list flatpaks in given repo
- `ostree summary --view --repo="/path/to/.ostree/repo"`: see ostree commit history

## Removing flatpaks from USB repo
The USB repo is a git-like folder contained in `.ostree/repo`. However, unlike git, this folder structure isn't
human-readable. There's a list of mirrors (e.g. `org.flathub.Stable`) in `.ostree/repo/refs/mirrors`, and each mirror
includes these folders: `app`, `appstream2`, and `runtime`. Within each mirror (e.g. `.ostree/repo/refs/mirrors/org.flathub.Stable/app/com.github.PintaProject.Pinta`)
there's an arch folder (e.g. `x86-64`), which contains a release branch file (e.g. `stable`), which contains a commit string. This string corresponds to files found in `.ostree/repo/objects`.
However, this isn't enough information to understand how to remove a flatpak from the .ostree repo.

See also:
- https://stackoverflow.com/a/51212880
- https://ostreedev.github.io/ostree/repository-management/

## Auto-adding USB repo
The systemd units in this repo are based on the following from [flatpak GitHub repo](https://github.com/flatpak/flatpak/tree/main/sideload-repos-systemd):
- `tmpfiles.d/*conf`: sets temp folder name and permissions
- `flatpak-sideload-usb-repo.service`: service file to set up symlinks
- `flatpak-create-sideload-symlinks.sh`: create & remove sideload-repo symlinks for all connected media devices
- `flatpak-sideload-usb-repo.path`: monitor `/media/$USER` for changes

### Enabling auto-detect of wasta-offline flatpaks
NOTE: This assumes that flatpaks have been exported to /media/$USER/<mount-dir>/wasta-offline/.ostree.
- Add (symlink) `install-files/libexec/wasta-offline-flatpak-setup-sideload-symlinks.sh` to `/usr/libexec/`
- Add (symlink) `install-files/tmpfiles.d/wasta-offline-flatpak-setup-sideload-repos.conf` to `/lib/tmpfiles.d/`
- Add (symlink) `install-files/user/wasta-offline-flatpak-setup-sideload-usb-repo.path` to `/usr/lib/systemd/user/`
- Add (symlink) `install-files/user/wasta-offline-flatpak-setup-sideload-usb-repo.service` to `/usr/lib/systemd/user/`
- `systemctl --user enable --now wasta-offline-flatpak-setup-sideload-usb-repo.path`
- `systemctl --user enable --now wasta-offline-flatpak-setup-sideload-usb-repo.service`
- `reboot # needed for tmpdir entry to take effect`
