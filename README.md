# Загрузка Linux

## Задания:
1. Попасть в систему без пароля несколькими способами.
2. Установить систему с LVM, после чего переименовать VG.
3. Добавить модуль в initrd.

### Попасть в систему без пароля несколькими способами.
Способ 1. init=/bin/sh:
- Во время отображения меню GRUB, нажать клавишу `e`, тем самым перейдя в режим редактирования параметров загрузки.
- В конце строки, начинающейся с `linux16`, добавить `init=/bin/sh`, нажать `ctrl+x` и дождаться приглашения ко вводу команд интерпретатора.
- Для внесения изменений в файловую систему, её необходимо перемонтировать с правами на запись: `mount -o remount,rw /`
> Примечания:
> - выставление параметра загрузки `init=/bin/sh`, указывает, что оболочка должна быть запущена сразу после загрузки ядра и initrd;
> - можно использовать другую оболочку, доступную в системе, к примеру `/bin/bash`.

Способ 2. rd.break:
- Во время отображения меню GRUB, нажать клавишу `e`, тем самым перейдя в режим редактирования параметров загрузки.
- В конце строки, начинающейся с `linux16`, добавить `rd.break`, нажать `ctrl+x` и дождаться приглашения ко вводу команд интерпретатора.
- Система загружается в аварийном (emergency) режиме (будет загружена с минимальным окружением).
- Для смены пароля необходимо:
  - перемонтировать ФС с правами на запись `mount -o remount,rw /sysroot`;
  - выполнить операцию изменения корневого каталога `chroot /sysroot`;
  - изменить пароль `passwd root`;
  - создать файл 'touch /.autorelabel'.

> Примечания:
> - `.autorelabel` - файл, позволяющий SELinux обновить метки всех файлов в этой системе на правильные контексты ([подробнее](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/security-enhanced_linux/sect-security-enhanced_linux-working_with_selinux-selinux_contexts_labeling_files)).
> - выставление параметра загрузки `rd.break`, останавливает процедуру загрузки, пока она еще находится в стадии initramfs (начальная файловая система в оперативной памяти).

Способ 3. rw init=/sysroot/bin/sh
- Во время отображения меню GRUB, нажать клавишу `e`, тем самым перейдя в режим редактирования параметров загрузки.
- В строке, начинающейся с `linux16`, заменить `ro` на ` rw init=/sysroot/bin/sh`, нажать `ctrl+x` и дождаться приглашения ко вводу команд интерпретатора.
- Система будет загружена в аварийном (emergency) режиме (с минимальным окружением).
- Для смены пароля необходимо:
  - выполнить операцию изменения корневого каталога `chroot /sysroot`;
  - изменить пароль `passwd root`;
  - создать файл 'touch /.autorelabel'.

### Установить систему с LVM, после чего переименовать VG.
Просмотр информации о текщих VG:
```
# vgs
VG #PV #LV #SN Attr VSize VFree
centos 1 2 0 wz--n- <14.80 0
```
Переименование VG:
```
# vgrename centos root
Volume group "centos" successfully renamed to "root"
```
Правка конфигурационных файлов: `/etc/fstab`, `/etc/default/grub`, `/boot/grub2/grub.cfg`. В них необходимо изменить старое имя VG "centos", на новое "root".

Переконфигурация initrd:
```
mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
```
Проверка после перезагрузки:
```
# vgs
VG #PV #LV #SN Attr VSize VFree
root 1 2 0 wz--n- <14.80 0
```

### Добавить модуль в initrd.
Создание каталога для модуля:
```
# mkdir /usr/lib/dracut/modules.d/01test
```
В каталог необходимо добавить скрипты [module-setup.sh](https://github.com/shulgazavr/ld_sys/blob/main/module-setup.sh) и [test.sh](https://github.com/shulgazavr/ld_sys/blob/main/test.sh).

Переконфигурация initrd:
```
# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
```
Проверка загруженности модуля:
```
# lsinitrd -m /boot/initramfs-$(uname -r).img | grep test
test
```
Для просмотра результата работы модуля, необходимо удалить `rhgb quiet` из загрузочной строки конфигурационного файла `/boot/grub2/grub.cfg`. 
> Примечание: удаление параметров `rhgb` и `quiet` увеличивает подробность загрузочных сообщений.
> 
Проверяем корректность введённых изменений:
```
# grub2-mkconfig
```
Переконфигурация grub:
```
/boot/grub2/grub.cfg (grub2-mkconfig | tee /boot/grub2/grub.cfg)
```
В итоге, при загрузке, будет пауза на 10 секунд и отображение пингвина.

