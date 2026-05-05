# Niri Config

Wayland compositor configuration for Fedora.

## Перенос на новое устройство

### 1. Установить зависимости

```bash
sudo dnf install niri alacritty fuzzel swaylock waybar dunst brightnessctl playerctl
```

### 2. Скопировать конфиг

```bash
git clone <your-repo-url> ~/.config/niri
```

### 3. Настроить мониторы

Откройте `config.kdl` и найдите секцию `output`. Раскомментируйте и настройте под своё железо:

```
# Узнать имена портов:
niri msg outputs

# Пример (раскомментировать в config.kdl):
output "HDMI-A-1" {
    mode "1920x1080@60"
    scale 1
    transform "normal"
    position x=0 y=0
}
```

Если работаете на ноутбуке без внешних мониторов — оставьте закомментированным, Niri сам определит встроенный дисплей.

### 4. Проверить конфиг

```bash
niri validate
```

### 5. Убедиться что программы доступны

Хоткеи используют следующие программы:

| Хоткей | Программа | Зачем |
|--------|-----------|-------|
| Mod+T | alacritty | Терминал |
| Mod+D | fuzzel | App launcher |
| Mod+Shift+S | alacritty + nvim | Редактор конфига |
| Super+Alt+L | swaylock | Блокировка экрана |
| spawn-at-startup | waybar | Панель |
| spawn-at-startup | dunst | Уведомления |
| XF86Audio* | wpctl / playerctl | Звук / медиа |
| XF86Mon* | brightnessctl | Яркость (ноутбук) |

Замените на аналоги по вкусу — редактируйте секцию `binds` в `config.kdl`.

### 6. Настроить запуск

Если niri не в display manager (gdm/sddm), создайте systemd user service:

```bash
mkdir -p ~/.config/systemd/user/
```

Файл `~/.config/systemd/user/niri.service`:

```ini
[Unit]
Description=Niri Wayland compositor
After=display-manager.service

[Service]
ExecStart=/usr/bin/niri --session
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

```bash
systemctl --user daemon-reload
systemctl --user enable --now niri.service
```
