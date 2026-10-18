# NeutrinoOS

**Mikelsoft Corporation INC.** — *Light as a neutrino, fast as light*

32-битная операционная система для x86, написанная с нуля. Своё ядро на C + NASM, собственные драйверы, FAT12 поверх ATA PIO, Win95-подобный GUI с перекрывающимися окнами и псевдо-мышью.

Текущая версия: **v0.9.2**

---

## Быстрый старт (Linux)

Самый простой способ запустить NeutrinoOS — скачать готовый архив с версией и запустить его. Всё уже собрано: ядро, ISO, образ диска.

### 1. Скачать архив

Зайди в раздел **Releases** этого репозитория и скачай последний архив, например:

```

NeutrinoOS_v0.9.2.zip

```

Если раздела Releases нет — качай ZIP всей ветки (`Code → Download ZIP`).

### 2. Распаковать

```bash
unzip NeutrinoOS_v0.9.2.zip -d NeutrinoOS
cd NeutrinoOS
```

Если у тебя `tar.gz` — соответственно `tar -xzf`.

### 3. Установить QEMU (один раз)

QEMU — это эмулятор, в котором запускается NeutrinoOS. Ставится в любом дистрибутиве:

**Debian / Ubuntu / Mint:**

```
sudo apt update
sudo apt install qemu-system-x86
```

**Arch / Manjaro:**

```
sudo pacman -S qemu
```

**Fedora:**

```
sudo dnf install qemu-system-x86
```

### 4. Запустить

```
chmod +x run.sh
./run.sh
```

Откроется окно QEMU с работающей NeutrinoOS. Всё.

---

## Что делать после запуска

### Загрузка

Через пару секунд появится прогресс-бар инициализации, потом MOTD, потом shell с приглашением:

```
user@neutrino>
```

Пароль по умолчанию: `1234`. Логин: `user`.
Но в стандартной сборке включён автовход — пароль не спросит.

### Полезные команды

```
HELP          — список всех команд
GUI           — открыть рабочий стол (Win95-стиль)
FETCH         — сводка о системе (как neofetch)
CPU           — информация о процессоре
DF            — занятость дисков
DIR           — список файлов на диске
FTE           — полноэкранный редактор
WRD           — текстовый процессор
SETTINGS      — настройки (юзер, пароль, тема, TZ)
SHUTDOWN      — выключить систему
```

### Управление в GUI

| Клавиша ↕▾ | Действие ↕▾ |
|---|---|
| −Стрелки | Перемещение курсора (псевдо-мышь) |
| −`[` | Левый клик в позиции курсора |
| −`]` | Правый клик |
| −`Alt+Del` | Открыть / закрыть Start menu |
| −`Tab` | Переключение фокуса между окнами |
| −`Esc` | Закрыть меню / активное окно |
| −`0-9 + - * /` | Ввод в активный калькулятор |
| −`C` | Очистить калькулятор |
| −`Enter` | `=` в калькуляторе |
⚙

### Как выключить

Либо в GUI: `Start → Shut Down`.
Либо в shell: команда `SHUTDOWN`.
Либо просто закрой окно QEMU — это как выдернуть питание, но диск FAT12 не пострадает.

---

## Если что-то не работает

### `./run.sh: команда не найдена`

Ты пытаешься запустить без `./`. В Linux текущая папка не в `$PATH`, поэтому нужно:

```
./run.sh
```

Или `bash run.sh`.

### `./run.sh: Permission denied`

Нет бита исполнения:

```
chmod +x run.sh
```

### `qemu-system-i386: command not found`

Не установлен QEMU. См. шаг 3 выше.

### `Could not access KVM kernel module`

Это нормально — NeutrinoOS работает и без KVM (просто чуть медленнее). QEMU выдаст warning, но запустится.

Если хочешь KVM (быстрее):

```
# Debian/Ubuntu
sudo apt install qemu-kvm
sudo usermod -aG kvm $USER
# перелогиниться
```

И раскомментировать/добавить в `run.sh` строку `-enable-kvm`.

### Окно QEMU открылось, но экран чёрный

Подожди 2–3 секунды — идёт инициализация. Если через 10 секунд всё ещё чёрный — попробуй:

```
qemu-system-i386 -cdrom neutrino.iso -m 32M -boot d
```

Без `-drive` (без диска). Система должна загрузиться в режиме ramdisk.

### `disk.img missing`

Образ диска потерялся. Пересобери через:

```
./build.sh
```

Или запусти без диска (см. выше) — тогда всё в оперативной памяти.

---

## Сборка из исходников (если хочешь)

Готовый архив уже собран, но если хочешь пересобрать сам — понадобится тулчейн:

```
# Debian / Ubuntu / Mint
sudo apt install gcc-multilib nasm xorriso mtools dosfstools \
                 grub-pc-bin grub-common qemu-system-x86

# Arch
sudo pacman -S gcc nasm xorriso mtools dosfstools grub qemu

# Fedora
sudo dnf install gcc nasm xorriso mtools dosfstools grub2-tools qemu-system-x86
```

Потом:

```
./build.sh              # полная сборка: kernel + disk.img + neutrino.iso
./build.sh --run        # сборка + запуск QEMU
./build.sh --test       # сборка + headless smoke-тест (5 сек)
./build.sh --clean      # очистка артефактов
```

---

## Что внутри

```
neutrino.iso       —
```

Конфиги на диске (внутри `disk.img`):

```
FIR
```

Если хочешь сбросить всё — пересобери `disk.img` через `./build.sh --clean && ./build.sh`.

---

## Ограничения

- **Только BIOS (legacy), не UEFI.** На современных ноутбуках с UEFI-only и Secure Boot не загрузится. В QEMU — работает всегда.
- **Только QEMU.** На VirtualBox/VMware не проверялось.
- **Нет реальной мыши.** Псевдо-мышь: стрелки + `[` / `]`.
- **Нет сети, нет USB, нет звука** кроме PC speaker.
- **FAT12 без подкаталогов и длинных имён.** FAT16 запланирован на 0.11.0.
- **Однозадачность.** Нет процессов, нет потоков, ring 0 only.

Это не баги — это границы текущей версии. См. `ROADMAP.md` для планов.

---

## Документация

- `README.md` — этот файл
- `CHANGELOG.md` — что менялось между версиями
- `ROADMAP.md` — план развития
- `RESTORE_POINT_v0.9.2.txt` — детальный снимок состояния v0.9.2
- `DEVELOPER_GUIDE.md` — для тех, кто хочет копать код

---

## Лицензия

MIT. Используй, форкай, ломай, чини — как хочешь.
Обратная связь — через Issues в этом репозитории.

**Mikelsoft Corporation INC.**
*«Light as a neutrino, fast as light»*

