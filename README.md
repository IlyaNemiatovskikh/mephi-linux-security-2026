# Домашнее задание № 1. Изучение средств защиты ОС GNU/Linux

- Бухгалтерский шифр: **374537**.
- ОС: Fedora Linux 44 (Server Edition), виртуальная машина VirtualBox.
- SELinux: Enforcing.

## 1. Создание пользователя

Создан пользователь `user1` с UID 1234 и дополнительной группой `students`. Домашний каталог: `/home/user1`, оболочка: `/bin/bash`.

Пароль действует 90 дней, предупреждение об истечении срока появляется за 7 дней. Проверка:

```bash
id user1
sudo chage -l user1
```

Результаты: [user1-id.out](user1-id.out), [user1-chage.out](user1-chage.out). [Скриншот](screenshots/01-user1.png).

## 2. Мониторинг файлов и процессов

Файлы с битом set-UID найдены командой:

```bash
sudo find / -type f -perm -4000 -print
```

Список сохранён в [suid-files.out](suid-files.out). В [privileged-processes.out](privileged-processes.out) найден процесс `passwd` с RUID 1234 и EUID 0.

[Скриншот](screenshots/02-processes.png).

## 3. Механизм set-UID

Копия `cat` находится в `/home/user1/lab-bin/cat-suid`, принадлежит `root:root` и имеет права `4755`.

Проверка от имени `user1`:

```bash
/usr/bin/cat /var/lib/mephi-security/root-only.txt
/home/user1/lab-bin/cat-suid /var/lib/mephi-security/root-only.txt
```

Обычный `cat` получил отказ в доступе, а `cat-suid` прочитал закрытый файл. Результаты: [до](suid-before.out) и [после](suid-after.out). [Скриншот](screenshots/03-suid.png).

## 4. Механизм capabilities

Копии `/home/user1/lab-bin/chown-cap` назначена capability `cap_chown=ep`.

Проверка от имени `user1`:

```bash
/usr/bin/chown root:root /home/user1/lab-data/owner-demo.txt
/home/user1/lab-bin/chown-cap root:root /home/user1/lab-data/owner-demo.txt
```

Обычный `chown` получил отказ, а `chown-cap` сменил владельца файла с `user1` на `root`. Результаты: [getcap.out](getcap.out), [cap-before.out](cap-before.out), [cap-owner-after.out](cap-owner-after.out). [Скриншот](screenshots/04-capabilities.png).

## 5. Механизм sudo

Правило в [etc/sudoers.d/90-user1-time](etc/sudoers.d/90-user1-time) разрешает `user1` менять системное время через `/usr/bin/timedatectl set-time` с датой в формате `YYYY-MM-DD HH:MM:SS`.

Во время опыта NTP отключался. От имени `user1` выполнена команда:

```bash
sudo /usr/bin/timedatectl set-time "$(date -d '+1 minute' '+%F %T')"
```

Команда завершилась с `exit=0`. Разрешения: [sudo-user1.out](sudo-user1.out), время [до](time-before.out) и [после](time-after.out). [Скриншот](screenshots/05-sudo-time.png).

## Проверка после перезагрузки

UID, группы и срок действия пароля сохранились. `cat-suid` снова прочитал закрытый файл, а `chown-cap` сменил владельца нового файла на `root:root`.

Результаты: [user1-id-after-reboot.out](user1-id-after-reboot.out), [user1-chage-after-reboot.out](user1-chage-after-reboot.out), [suid-after-reboot.out](suid-after-reboot.out), [cap-after-reboot.out](cap-after-reboot.out). [Скриншот](screenshots/06-after-reboot.png).

Правило sudo сохранилось, что видно в [sudo-after-reboot.out](sudo-after-reboot.out). Повторная смена времени после перезагрузки не зафиксирована.

## Материалы работы

- [mephi-screenshot.png](mephi-screenshot.png) - снимок с бухгалтерским шифром.
- [etc/](etc/): копии `passwd`, `group`, `shadow`, `sudoers` и правила sudo из учебной VM.
- [stat.out](stat.out) и [getcap.out](getcap.out) - права файлов и capability.
- [history.out](history.out) - сводная история нескольких сеансов.
- [screenshots/](screenshots/) - снимки основных этапов работы.
