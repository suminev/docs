---
description: "Network Time Protocol Servers (NTP) – позволяет выполнять настройку синхронизации с общедоступными NTP серверами времени на виртуальных машинах {{ yandex-cloud }} под управлением Windows Server и Linux. Описан порядок настройки серверов для синхронизации."
keywords:
  - ntp
  - ntp server
  - sntp
  - синхронизация времени
  - нтп сервер
  - ntp сервер
---

# Настройка синхронизации часов с помощью NTP

На [виртуальных машинах](../../compute/concepts/vm.md) {{ compute-full-name }} можно настроить синхронизацию времени с общедоступными серверами по протоколу [NTPv4](https://tools.ietf.org/html/rfc5905).

Время ВМ синхронизируется, чтобы корректно взаимодействовать с другими серверами и приложениями, обрабатывать данные в реальном времени, хранить точную информацию о времени в системных логах и пр.

Рекомендуемые NTP-серверы для синхронизации:
* `ntp1.yandex.net`;
* `ntp2.yandex.net`;
* `ntp0.NL.net`;
* `ntp2.vniiftri.ru`;
* `ntp.ix.ru`;
* `ntps1-1.cs.tu-berlin.de`.

{% note info %}

Список рекомендуемых серверов может меняться. О том, что вам нужно внести изменения в конфигурацию ВМ, {{ yandex-cloud }} сообщит за 72 часа.

{% endnote %}

Чтобы настроить синхронизацию времени:
1. [Подготовьте облако к работе](#before-you-begin).
1. [Подготовьте инфраструктуру](#prepare-infrastructure).
1. [Настройте синхронизацию](#setup-ntp).
1. [Проверьте синхронизацию](#check-ntp).

Если созданные ресурсы вам больше не нужны, [удалите их](#clear-out).

## Перед началом работы {#before-you-begin}

{% include [before-you-begin](../_tutorials_includes/before-you-begin.md) %}

### Необходимые платные ресурсы {#paid-resources}

В стоимость синхронизации часов с помощью NTP входит плата за вычислительные ресурсы и диски ВМ (см. [тарифы {{ compute-full-name }}](../../compute/pricing.md)).

## Подготовьте инфраструктуру {#prepare-infrastructure}

1. Если у вас еще нет ВМ, [создайте](../../compute/operations/#vm-create) ее.
1. [Привяжите](../../compute/operations/vm-control/vm-attach-public-ip.md) к ВМ публичный IP-адрес.

## Настройте синхронизацию {#setup-ntp}

{% list tabs group=operating_system %}

- Linux (systemd) {#linux-systemd}

  На ВМ с ОС на основе Linux должен быть включен DHCP-клиент с опцией `42` (`Network Time Protocol Servers`). DHCP-клиент автоматически применяет список серверов для синхронизации времени, который получает от DHCP-сервера. В образах {{ compute-name }} ОС уже настроены нужным образом.

  На случай если DHCP-сервер будет недоступен, укажите запасные серверы синхронизации времени в настройках системы.

  {% note info %}

  Сервис `systemd-timesyncd` может конфликтовать с сервисом `ntpd`, если они запущены одновременно. Чтобы посмотреть запущенные сервисы, используйте команду `ps -eF`. Вы можете либо удалить сервис `ntpd`, либо настроить синхронизацию времени с его помощью, см. вкладку **Linux (ntpd)**. 

  {% endnote %}

  Чтобы настроить синхронизацию времени:
  1. [Подключитесь](../../compute/operations/vm-connect/ssh.md#vm-connect) к ВМ по SSH.
  1. Откройте файл `/etc/systemd/timesyncd.conf`, для этого в терминале выполните команду:

      ```bash
      sudo nano /etc/systemd/timesyncd.conf
      ```

  1. Укажите адреса рекомендуемых серверов в секции `[Time]` в параметре `FallbackNTP=`, например:
     
      
      ```text
      FallbackNTP=ntp1.yandex.net ntp2.yandex.net ntp0.NL.net ntp2.vniiftri.ru ntp.ix.ru ntps1-1.cs.tu-berlin.de
      ```



  1. В конфигурационном файле `systemd.network` укажите параметр `UseNTP=true`. Конфигурационный файл обычно размещен в директориях `/etc/systemd/network/` или `/var/lib/systemd/network/`.

      Также может использоваться конфигурационный файл `networkd.conf` в директории `/etc/systemd/`.
  1. Перезапустите сервис синхронизации времени:

      ```bash
      sudo systemctl restart systemd-timesyncd
      ```

- Linux (ntpd) {#linux-ntpd}

  На ВМ с ОС на основе Linux должен быть включен DHCP-клиент с опцией `42` (`Network Time Protocol Servers`). DHCP-клиент автоматически применяет список серверов для синхронизации времени, который получает от DHCP-сервера. В образах {{ compute-name }} ОС уже настроены нужным образом.

  На случай если DHCP-сервер будет недоступен, укажите запасные серверы синхронизации времени в настройках системы.

  {% note info %}

  Сервис `ntpd` может конфликтовать с сервисом `systemd-timesyncd`, если они запущены одновременно. Чтобы посмотреть запущенные сервисы, используйте команду `ps -eF`. Вы можете либо удалить сервис `systemd-timesyncd`, либо настроить синхронизацию времени с его помощью, см. вкладку **Linux (systemd)**. 

  {% endnote %}

  Чтобы настроить синхронизацию времени:
  1. [Подключитесь](../../compute/operations/vm-connect/ssh.md#vm-connect) к ВМ по SSH.
  1. Установите сервис `ntpd`, для этого в терминале выполните команду:

      ```bash
      sudo apt-get install ntp
      ```

  1. Откройте файл `/etc/ntp.conf`:

      ```bash
      sudo nano /etc/ntp.conf
      ```

  1. Укажите адреса рекомендуемых серверов. Адреса серверов по умолчанию закомментируйте символом `#` в начале строки, например:
     
      
      ```text
      # Use servers from the NTP Pool Project. Approved by Ubuntu Technical Board
      # on 2011-02-08 (LP: #104525). See http://www.pool.ntp.org/join.html for
      # more information.
      # server 0.ubuntu.pool.ntp.org
      # server 1.ubuntu.pool.ntp.org
      # server 2.ubuntu.pool.ntp.org
      # server 3.ubuntu.pool.ntp.org
      server ntp1.yandex.net
      server ntp2.yandex.net
      server ntp0.NL.net
      server ntp2.vniiftri.ru
      server ntp.ix.ru
      server ntps1-1.cs.tu-berlin.de
      ```



  1. Перезапустите сервис синхронизации времени:

      ```bash
      sudo service ntp restart
      ```

- Windows Server {#windows}

  Чтобы настроить синхронизацию времени:
  1. [Подключитесь](../../compute/operations/vm-connect/rdp.md) к ВМ по RDP.
  1. Запустите командную строку cmd или PowerShell от имени администратора.
  1. Остановите службу Windows Time, для этого выполните команду:

      ```powershell
      net stop w32time
      ```

      Результат:

      ```powershell
      The Windows Time service is stopping.
      The Windows Time service was stopped successfully.
      ```

  1. Укажите адреса рекомендуемых серверов, например:

      

      ```powershell
      w32tm /config /syncfromflags:manual /manualpeerlist:"ntp1.yandex.net ntp2.yandex.net ntp0.NL.net ntp2.vniiftri.ru ntp.ix.ru ntps1-1.cs.tu-berlin.de"
      ```



      Результат:

      ```powershell
      The command completed successfully.
      ```

  1. Укажите, что ВМ является надежным источником времени:

      ```powershell
      w32tm /config /reliable:yes
      ```

      Результат:

      ```powershell
      The command completed successfully.
      ```

  1. Запустите службу Windows Time:

      ```powershell
      net start w32time
      ```

      Результат:

      ```powershell
      The Windows Time service is starting.
      The Windows Time service was started successfully.
      ```

{% endlist %}

## Проверьте синхронизацию {#check-ntp}

{% list tabs group=operating_system %}

- Linux (systemd) {#linux-systemd}

  Выполните команду:

  ```bash
  timedatectl
  ```

  Результат:

  ```bash
  ...
  System clock synchronized: yes                        
                NTP service: active   
  ...
  ```

- Linux (ntpd) {#linux-ntpd}

  Выполните команду:

  ```bash
  sudo ntpq -p
  ```

  Результат:

  
  ```bash
      remote           refid      st t when poll reach   delay   offset  jitter
  ==============================================================================
  ntp.ubuntu.com  .POOL.          16 p    -   64    0    0.000   +0.000   0.000
  indigoberry.yan .INIT.          16 u    -   64    0    0.000   +0.000   0.000
  ivoryberry.yand .INIT.          16 u    -   64    0    0.000   +0.000   0.000
  ntp0.nl.uu.net  .GPS.            1 u    1   64    1   47.692   -8.929   0.000
  ntp2.vniiftri.r .MRS.            1 u    1   64    1   10.946   -8.960   0.000
  ntp.ix.ru       .GLN.            1 u    5   64    1    7.266   -8.459   0.000
  ntps1-1.eecsit. .GPS.            1 u    2   64    1   32.314   -8.960   0.000
  alphyn.canonica 132.163.96.1     2 u    4   64    1  121.368   -9.121   0.000
  185.125.190.58  86.23.195.30     2 u    3   64    1   52.084   -8.337   0.000
  185.125.190.56  194.121.207.249  2 u    1   64    1   53.791   -8.268   0.000
  ```



- Windows Server {#windows}

  Выполните команду, указав имя ВМ:

  ```powershell
  w32tm /query /configuration
  ```

  Результат:

  
  ```powershell
  ...
  [TimeProviders]
  ...
  NtpServer: ntp1.yandex.net ntp2.yandex.net ntp0.NL.net ntp2.vniiftri.ru ntp.ix.ru ntps1-1.cs.tu-berlin.de (Local)
  ...
  ```



{% endlist %}

## Как удалить созданные ресурсы {#clear-out}

Чтобы перестать платить за созданные ресурсы [удалите](../../compute/operations/vm-control/vm-delete.md) ВМ.
