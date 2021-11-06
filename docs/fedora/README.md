# Fedora

[https://getfedora.org/en/workstation/download/](https://getfedora.org/en/workstation/download/)

[5 Things You MUST DO After Installing Fedora 35](https://www.youtube.com/watch?v=-NwWE9YFFIg)

- [00:00](https://www.youtube.com/watch?v=-NwWE9YFFIg&t=0s)  Fedora Introduction
- [01:05](https://www.youtube.com/watch?v=-NwWE9YFFIg&t=65s) 1. Optimize DNF Config

```
sudo nano /etc/dnf/dnf.conf
fastestmirror=True
max_parallel_downloads=10
defaultyes=True
```

- [03:20](https://www.youtube.com/watch?v=-NwWE9YFFIg&t=200s) 2. System Update

```
sudo dnf update
```

- [04:00](https://www.youtube.com/watch?v=-NwWE9YFFIg&t=240s) 3. Enable RPM [Fedora Docs RPM Update Link](https://docs.fedoraproject.org/en-US/quick-docs/setup_rpmfusion/)

```
sudo dnf install https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
sudo dnf install https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

- [06:16](https://www.youtube.com/watch?v=-NwWE9YFFIg&t=376s) 4. Installing Media Codecs [Media plugins Link](https://docs.fedoraproject.org/en-US/quick-docs/assembly_installing-plugins-for-playing-movies-and-music/)

```
sudo dnf install gstreamer1-plugins-{bad-\*,good-\*,base} gstreamer1-plugin-openh264 gstreamer1-libav --exclude=gstreamer1-plugins-bad-free-devel
sudo dnf install lame\* --exclude=lame-devel
sudo dnf group upgrade --with-optional Multimedia
```

- [08:00](https://www.youtube.com/watch?v=-NwWE9YFFIg&t=480s) 5. Install Extensions, Software, etc.
    - [Install Chromium Link](https://docs.fedoraproject.org/en-US/quick-docs/installing-chromium-or-google-chrome-browsers/)

```
sudo dnf install chromium
sudo dnf update chromium
```

