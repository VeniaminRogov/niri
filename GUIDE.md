# Гайд по установке Niri на Fedora 43

Пошаговая инструкция по развёртыванию Niri Wayland-композитора с тем же стеком инструментов, что и в этом репозитории.

## 1. Предварительные требования

- **Fedora Linux 43** (Workstation Edition или Server)
- **GDM** как display manager (установлен по умолчанию в Workstation)
- Dual HDMI мониторы 1920×1080 (настройка под ваш hardware см. раздел 4)
- Шрифт [Agave Nerd Font Mono](https://github.com/ryanoasis/nerd-fonts)

## 2. Установка пакетов

### Базовые утилиты

```bash
sudo dnf install -y \
  niri \
  fuzzel \
  alacritty \
  swaylock \
  dunst \
  brightnessctl \
  playerctl \
  curl \
  jq
```

### Шрифты

```bash
sudo dnf install -y "nerd-fonts-agave*"
```

Если пакет недоступен, скачайте вручную:

```bash
git clone --depth 1 --filter=blob:none --sparse https://github.com/ryanoasis/nerd-fonts.git ~/.local/share/fonts/nerd-fonts
cd ~/.local/share/fonts/nerd-fonts
git sparse-checkout set "patched/NerdFonts/Addons/Agans/"
fc-cache -fv
cd ~
```

### Pandora (wallpaper daemon)

```bash
cargo install pandora
```

### VibePanel (опционально, вместо waybar + mako)

```bash
git clone https://github.com/prankstr/vibepanel.git
cd vibepanel
cargo build --release
cp target/release/vibepanel ~/.local/bin/
cd ~
```

## 3. Настройка Niri

Создайте конфиг:

```bash
mkdir -p ~/.config/niri
```

Скопируйте `config.kdl` из этого репозитория или создайте с нуля. Минимальный конфиг для dual-monitor:

```kdl
// ~/.config/niri/config.kdl
input {
    keyboard {
        xkb {
            layout "us,ru"
            options "grp:win_space_toggle,compose:ralt,ctrl:nocaps"
        }
        numlock
    }
    touchpad {
        tap
        natural-scroll
    }
}

// Левый монитор
output "HDMI-A-2" {
    mode "1920x1080@74.973"
    scale 1
    position x=-1920 y=0
}

// Правый монитор (основной)
output "HDMI-A-1" {
    mode "1920x1080@74.973"
    scale 1
    position x=0 y=0
}

layout {
    gaps 16
    default-column-width { proportion 0.5; }
    focus-ring {
        width 4
        active-color "#7fc8ff"
        inactive-color "#505050"
    }
}

spawn-at-startup "vibepanel"
spawn-at-startup "dunst"

screenshot-path "~/Pictures/Screenshots/Screenshot from %Y-%m-%d %H-%M-%S.png"

hotkey-overlay {
    skip-at-startup
}

binds {
    Mod+T { spawn "alacritty"; }
    Mod+D { spawn "fuzzel"; }
    Super+Alt+L { spawn "swaylock"; }

    XF86AudioRaiseVolume allow-when-locked=true { spawn-sh "wpctl set-volume @DEFAULT_AUDIO_SINK@ 0.1+ -l 1.0"; }
    XF86AudioLowerVolume allow-when-locked=true { spawn-sh "wpctl set-volume @DEFAULT_AUDIO_SINK@ 0.1-"; }
    XF86AudioMute allow-when-locked=true { spawn-sh "wpctl set-mute @DEFAULT_AUDIO_SINK@ toggle"; }
    XF86AudioMicMute allow-when-locked=true { spawn-sh "wpctl set-mute @DEFAULT_AUDIO_SOURCE@ toggle"; }

    XF86AudioPlay allow-when-locked=true { spawn-sh "playerctl play-pause"; }
    XF86AudioStop allow-when-locked=true { spawn-sh "playerctl stop"; }
    XF86AudioPrev allow-when-locked=true { spawn-sh "playerctl previous"; }
    XF86AudioNext allow-when-locked=true { spawn-sh "playerctl next"; }

    Mod+Q { close-window; }
    Mod+Left  { focus-column-left; }
    Mod+Down  { focus-window-down; }
    Mod+Up    { focus-window-up; }
    Mod+Right { focus-column-right; }
    Mod+H { focus-column-left; }
    Mod+J { focus-window-down; }
    Mod+K { focus-window-up; }
    Mod+L { focus-column-right; }

    Mod+1 { focus-workspace 1; }
    Mod+2 { focus-workspace 2; }
    Mod+3 { focus-workspace 3; }
    Mod+4 { focus-workspace 4; }
    Mod+5 { focus-workspace 5; }
    Mod+6 { focus-workspace 6; }
    Mod+7 { focus-workspace 7; }
    Mod+8 { focus-workspace 8; }
    Mod+9 { focus-workspace 9; }

    Mod+Shift+E { quit; }
    Ctrl+Alt+Delete { quit; }

    Print { screenshot; }
}
```

Полный конфиг с комментариями см. в `~/.config/niri/config.kdl` на рабочем устройстве.

### Подстройка под свой hardware

Чтобы узнать имена ваших мониторов:

```bash
niri msg outputs
```

Чтобы узнать доступные режимы, посмотрите вывод `niri msg outputs` в формате JSON. Замените `HDMI-A-1`/`HDMI-A-2` и `mode` на ваши значения.

## 4. Настройка VibePanel

```bash
mkdir -p ~/.config/vibepanel
```

### config.toml

```toml
[widgets]
right = ["tray", "updates", "cpu", "quick_settings", "custom-input", "clock", "notifications"]

[widgets.custom-input]
icon = "input-keyboard-symbolic"
exec = "~/.local/bin/vibe-input-status"
template = "{output}"
interval = 2
tooltip = "Keyboard layout"

[theme]
mode = "dark"

[advanced]
compositor = "niri"
```

### style.css (Tokyo Night)

```css
/* ~/.config/vibepanel/style.css */
* {
  --color-foreground-primary: #c0caf5;
  --color-foreground-muted: #a9b1d6;
  --color-foreground-disabled: #565f89;
  --color-foreground-faint: #3b4261;

  --color-accent-primary: #7aa2f7;
  --color-accent-subtle: #3d59a7;
  --color-accent-text: #7aa2f7;
  --color-accent-hover-bg: #4564c2;

  --color-card-overlay: rgba(26, 27, 38, 0.85);
  --color-card-overlay-hover: rgba(45, 48, 68, 0.85);
  --color-card-overlay-subtle: rgba(20, 21, 30, 0.85);
  --color-card-overlay-strong: rgba(30, 32, 48, 0.9);

  --color-border-subtle: #3b4261;
  --color-widget-hover-bg: rgba(45, 48, 68, 0.5);

  --color-background-bar: transparent;

  --font-family: "Agave Nerd Font", monospace;
  --font-size: 16px;
}
```

### Скрипт раскладки клавиатуры

```bash
mkdir -p ~/.local/bin
```

```bash
#!/bin/bash
# ~/.local/bin/vibe-input-status
if [[ "$1" == "--toggle" ]]; then
  niri msg action switch-layout > /dev/null 2>&1
  sleep 0.05
fi

layout=$(niri msg keyboard-layouts 2>&1 | grep '*' | awk '{for(i=3;i<=NF;i++) printf "%s ", $i; print ""}' | xargs)

case "$layout" in
  *Russian*) echo "RU" ;;
  *English*) echo "US" ;;
  *) echo "$layout" ;;
esac
```

```bash
chmod +x ~/.local/bin/vibe-input-status
```

## 5. Настройка Alacritty

```bash
mkdir -p ~/.config/alacritty
```

Минимальный конфиг с Tokyo Night:

```toml
# ~/.config/alacritty/alacritty.toml
[font]
size = 12

[font.normal]
family = "Agave Nerd Font Mono"
style = "Regular"

[window]
decorations = "None"

[colors.primary]
background = "#1a1b26"
foreground = "#a9b1d6"

[colors.normal]
black = "#15161e"
red = "#f7768e"
green = "#9ece6a"
yellow = "#e0af68"
blue = "#7aa2f7"
magenta = "#bb9af7"
cyan = "#7dcfff"
white = "#a9b1d6"
```

## 6. Настройка Dunst

```bash
mkdir -p ~/.config/dunst
```

```ini
# ~/.config/dunst/dunstrc
[global]
origin = top-right
offset = 12x30
width = 380
height = 400
padding = 20
corner_radius = 12
font = Agave Nerd Font 16
notification_limit = 5
show_indicators = false
icon_position = left
max_icon_size = 48

[urgency_normal]
background = #1a1b26
foreground = #c0caf5
timeout = 5

[urgency_critical]
background = #1a1b26
foreground = #f7768e
timeout = 0

[urgency_low]
background = #1a1b26
foreground = #565f89
timeout = 3
```

## 7. Настройка Fuzzel

```bash
mkdir -p ~/.config/fuzzel
```

Скопируйте пример:

```bash
fuzzel --print-config > ~/.config/fuzzel/config
```

Отредактируйте шрифт и цвета под Tokyo Night при необходимости.

## 8. Autostart сервисы

### Dunst через systemd user

```bash
mkdir -p ~/.config/systemd/user/
```

```ini
# ~/.config/systemd/user/dunst.service
[Unit]
Description=Dunst notification daemon
After=graphical-session.target
Wants=dunst.service

[Service]
Type=simple
Environment=WLR_OUTPUT_NAME=HDMI-A-1
ExecStart=/usr/bin/dunst
Restart=on-failure
RestartSec=1

[Install]
WantedBy=graphical-session.target
```

```bash
systemctl --user daemon-reload
systemctl --user enable --now dunst.service
```

### Pandora (wallpaper)

Если используете Pandora для обоев, добавьте в `config.kdl`:

```kdl
spawn-at-startup "~/.cargo/bin/pandora" "daemon"

layer-rule {
    match namespace="^pandora$"
    place-within-backdrop true
}
```

## 9. Запуск Niri

### Через GDM (рекомендуется)

1. На экране входа выберите `Niri` рядом с именем пользователя
2. Войдите

### Через TTY

```bash
exec niri
```

Добавьте в `~/.xprofile` для автоматического запуска при выборе графического таргета:

```bash
# ~/.xprofile
export XDG_CURRENT_DESKTOP=niri
```

## 10. Проверка

После входа в Niri:

```bash
# Проверить состояние композитора
niri msg status

# Узнать имена мониторов
niri msg outputs

# Перезагрузить конфиг без выхода
niri msg config reload

# Проверить раскладку
niri msg keyboard-layouts
```

Убедитесь что:
- Оба монитора активны с правильным разрешением
- VibePanel отображается с widget'ами справа
- Fuzzel открывается по `Mod+D`
- Alacritty открывается по `Mod+T`
- swaylock работает по `Super+Alt+L`
- Горячие клавиши громкости/яркости работают
- Dunst показывает уведомления

## 11. Устранение проблем

### Мониторы не detected

```bash
niri msg outputs
```

Если мониторы не определены, проверьте кабель и попробуйте вручную указать mode:

```bash
niri msg outputs --raw  # подробный JSON
```

### VibePanel не запускается

```bash
~/.local/bin/vibepanel 2>&1 | head -20
```

Проверьте что `compositor = "niri"` в `[advanced]` секции config.toml.

### Клавиатура не переключается

Убедитесь что `options "grp:win_space_toggle"` в `input.keyboard.xkb` и нет конфликта с `Mod+Space` в binds секции.

### swaylock не показывает обои

```bash
swaylock -i ~/path/to/wallpaper.png
```

Для автоматической работы с Pandora, проверьте что swaylock запущен после загрузки обоев.

### Двойное переключение раскладки

Если `Mod+Space` переключает раскладку дважды, проверьте что в xkb options нет `grp:space_caps` или других conflict-опций.
