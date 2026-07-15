# Домашнє завдання №3 — Процеси та ресурси Linux

**ПІБ:** Ярцев Олексій

---

## Завдання 1. Огляд активних процесів

### 1.1. Виведення переліку всіх процесів системи

Команда:
```bash
ps aux
```

Приклад виконання:
```text
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  8.9  0.9  47744 40368 ?        Ss   15:30   0:01 /usr/bin/python3 /usr/bin/supervisord -n -c /etc/supervisord.conf
root         3  1.3  2.2 3799036 93420 ?       Sl   15:30   0:00 log_forwarder --service-name logrotate_loop -- /bin/bash /usr/local/init_scripts/logrotate_loop.sh
root       133  0.7  0.1  12512  6008 ?        S    15:30   0:00 /bin/bash /usr/local/init_scripts/logrotate_loop.sh
root       266  0.5  0.1  11036  4800 ?        S    15:30   0:00 sleep 86400
root      1270  2.1  1.6 3799036 71260 ?       Sl   15:30   0:00 log_forwarder --service-name python_tool -- /bin/bash /usr/local/init_scripts/python_tool.sh
root      1403  2.4  2.3 3799036 98404 ?       Sl   15:30   0:00 log_forwarder --service-name terminal_server -- /bin/bash /usr/local/init_scripts/terminal_server.sh
root      1629  0.2  0.1  12512  5884 ?        S    15:30   0:00 /bin/bash /usr/local/init_scripts/python_tool.sh
oai       1668  0.7  0.3  27612 14672 ?        Sl   15:30   0:00 tini -- /opt/pyvenv-python-tool/bin/python -m uvicorn --host 0.0.0.0 --port 8080 --log-config /opt/python-tool/uvicorn_logging.config jupyter_server.app:app
root      1785  0.0  0.1  12512  4676 ?        S    15:30   0:00 /bin/bash /usr/local/init_scripts/terminal_server.sh
oai       1806  0.2  0.1  12512  4444 ?        S    15:30   0:00 /bin/bash /opt/terminal-server/scripts/start-server.sh
oai       1820 27.7  2.7 165648 113676 ?       Sl   15:30   0:02 /opt/pyvenv-python-tool/bin/python -m uvicorn --host 0.0.0.0 --port 8080 --log-config /opt/python-tool/uvicorn_logging.config jupyter_server.app:app
oai       1907  7.3  1.3  65416 58252 ?        S    15:30   0:00 /opt/terminal-server/pyvenv/bin/python /opt/terminal-server/openai/server.py
oai       1912 50.2  7.5 2622840 315948 ?      Rsl  15:30   0:03 /opt/pyvenv/bin/python -m ipykernel_launcher -f /tmp/tmpa5fju1pq.json
oai       1990 79.8 10.3 74048760 432740 ?     Rsl  15:30   0:04 /opt/pyvenv/lib/python3.13/site-packages/artifact_tool/bin/artifact_tool_rpc_daemon-bun
...
```

**Пояснення:** `ps aux` виводить повний перелік процесів, що виконуються у системі, незалежно від того, якому користувачу вони належать і чи прив'язані до терміналу. У виводі відображаються PID, відсоток завантаження CPU та пам'яті, стан процесу і повна команда запуску.

---

### 1.2. Запуск `top` / `htop` для пошуку процесу з найбільшим споживанням RAM

Команда для інтерактивного режиму:
```bash
top
```

Щоб відсортувати список за обсягом використаної пам'яті, у `top` натискаємо `Shift + M`.

Для фіксації результату без інтерактивного режиму зручніше скористатись:
```bash
ps aux --sort=-%mem | head -n 6
```

Приклад результату:
```text
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
oai       1990 80.4 10.3 74048760 435544 ?     Rsl  15:30   0:04 /opt/pyvenv/lib/python3.13/site-packages/artifact_tool/bin/artifact_tool_rpc_daemon-bun
oai       1912 50.3  7.5 2622840 315976 ?      Rsl  15:30   0:03 /opt/pyvenv/bin/python -m ipykernel_launcher -f /tmp/tmpa5fju1pq.json
oai       1820 27.5  2.7 165648 113676 ?       Sl   15:30   0:02 /opt/pyvenv-python-tool/bin/python -m uvicorn --host 0.0.0.0 --port 8080 --log-config /opt/python-tool/uvicorn_logging.config jupyter_server.app:app
root      1403  2.4  2.3 3799036 98404 ?       Sl   15:30   0:00 log_forwarder --service-name terminal_server -- /bin/bash /usr/local/init_scripts/terminal_server.sh
root         3  1.3  2.2 3799036 93420 ?       Sl   15:30   0:00 log_forwarder --service-name logrotate_loop -- /bin/bash /usr/local/init_scripts/logrotate_loop.sh
```

**Пояснення:** ключ `--sort=-%mem` сортує список процесів за спаданням споживання пам'яті, а `head -n 6` залишає лише перші рядки (заголовок і п'ять найбільш ресурсомістких процесів). Найбільше пам'яті на момент перевірки споживав процес `artifact_tool_rpc_daemon-bun` (PID 1990).

---

### 1.3. Визначення PID поточної оболонки bash/zsh

Команда:
```bash
echo $$
ps -p $$ -o pid,ppid,comm,args
```

Приклад результату:
```text
PID поточної оболонки: 2013

  PID  PPID COMMAND  ARGS
 2013  2012 bash     bash -l
```

**Пояснення:** змінна `$$` містить PID процесу поточної оболонки. Команда `ps -p $$` виводить детальну інформацію саме про цей процес — його PID, PID батьківського процесу (PPID), назву команди та повний рядок запуску.

---

## Завдання 2. Робота у фоні та керування процесами

### 2.1. Запуск довготривалої команди у фоновому режимі

Команда:
```bash
sleep 1000 &
```

Приклад виконання:
```text
Запущено sleep 1000 у фоні. PID: 2022
```

**Пояснення:** символ `&` наприкінці команди переводить її виконання у фон, повертаючи керування терміналом одразу, не чекаючи завершення `sleep`.

---

### 2.2. Перевірка переліку фонових завдань

Команда:
```bash
jobs -l
```

Приклад результату:
```text
[1]+  2022 Running                 sleep 1000 &
```

**Пояснення:** `jobs -l` показує всі активні фонові завдання поточної сесії оболонки разом із їхніми PID та станом виконання.

---

### 2.3. Повернення фонового процесу на передній план

Команда:
```bash
fg %1
```

Щойно процес опиниться на передньому плані, його можна призупинити комбінацією:
```text
Ctrl + Z
```

**Пояснення:** `fg %1` активує перше фонове завдання й робить його знову "переднім" (foreground), тобто термінал очікуватиме на його завершення. `Ctrl + Z` призупиняє процес, переводячи його у стан "Stopped".

---

### 2.4. Примусове завершення процесу командою kill

Команда:
```bash
kill -9 <PID>
```

Приклад виконання:
```text
Процес 2022 завершено командою kill -9
```

**Пояснення:** сигнал `-9` (SIGKILL) миттєво й безумовно завершує процес, не даючи йому можливості коректно закритися самостійно.

---

### 2.5. Запуск команди через nohup

Команда:
```bash
nohup sleep 1000 > nohup_sleep.log 2>&1 &
```

Приклад виконання:
```text
nohup sleep 1000 запущено. PID: 2023
  PID  PPID STAT COMMAND         COMMAND
 2023  2013 S    sleep           sleep 1000
```

**Пояснення:** `nohup` захищає процес від завершення при закритті терміналу (ігнорує сигнал `HUP`), а перенаправлення `> nohup_sleep.log 2>&1` записує весь стандартний вивід і помилки у вказаний лог-файл.

---

## Завдання 3. Пріоритети та обмеження

### 3.1. Запуск команди з підвищеним значенням nice (нижчим пріоритетом)

Команда:
```bash
nice -n 10 sleep 1000 &
```

Приклад виконання:
```text
  PID  NI PRI STAT COMMAND         COMMAND
 2025  10   9 SN   sleep           sleep 1000
```

**Пояснення:** чим вище значення `nice` (діапазон від -20 до 19), тим нижчий пріоритет отримує процес у черзі планувальника, тобто він поступатиметься процесорним часом іншим процесам.

---

### 3.2. Зміна пріоритету вже запущеного процесу

Команда:
```bash
renice 15 -p <PID>
```

Приклад виконання:
```text
2025 (process ID) old priority 10, new priority 15
  PID  NI PRI STAT COMMAND         COMMAND
 2025  15   4 SN   sleep           sleep 1000
```

**Пояснення:** `renice` дозволяє змінити значення `nice` уже для процесу, який виконується, без необхідності його перезапуску.

---

### 3.3. Перегляд поточних обмежень ресурсів користувача

Команда:
```bash
ulimit -a
```

Приклад результату:
```text
real-time non-blocking time  (microseconds, -R) unlimited
core file size              (blocks, -c) unlimited
data seg size               (kbytes, -d) unlimited
scheduling priority                 (-e) 0
file size                   (blocks, -f) unlimited
pending signals                     (-i) 0
max locked memory           (kbytes, -l) 64
max memory size             (kbytes, -m) unlimited
open files                          (-n) 1048576
pipe size                (512 bytes, -p) 8
POSIX message queues         (bytes, -q) 819200
real-time priority                  (-r) 0
stack size                  (kbytes, -s) 8192
cpu time                   (seconds, -t) unlimited
max user processes                  (-u) unlimited
virtual memory              (kbytes, -v) unlimited
file locks                          (-x) unlimited
```

**Пояснення:** `ulimit -a` виводить усі поточні обмеження, встановлені для оболонки та процесів, які вона запускає — максимальний розмір файлу, кількість відкритих файлових дескрипторів, кількість процесів користувача тощо.

---

## Завдання 4. Моніторинг ресурсів

### 4.1. Виведення інформації про використання дискового простору

Команда:
```bash
df -h
```

Приклад результату:
```text
Filesystem      Size  Used Avail Use% Mounted on
none            8.0E  5.1M  8.0E   1% /
none            252G     0  252G   0% /dev
none             64M     0   64M   0% /dev/shm
none            1.0M   12K 1012K   2% /etc/hosts
none            2.0T  191G  1.8T  10% /caas_toolbox
none            1.0M   12K 1012K   2% /etc/hostname
none            252G     0  252G   0% /sys/fs/cgroup
none            1.0M   12K 1012K   2% /etc/resolv.conf
```

**Пояснення:** ключ `-h` ("human-readable") виводить розміри у зручному для читання форматі (K, M, G, T) замість блоків, показуючи загальний обсяг, використане та вільне місце для кожної змонтованої файлової системи.

---

### 4.2. Виведення обсягу вільної та зайнятої оперативної пам'яті

Команда:
```bash
free -h
```

Приклад результату:
```text
               total        used        free      shared  buff/cache   available
Mem:           4.0Gi       695Mi       3.3Gi          0B       251Mi       3.3Gi
Swap:             0B          0B          0B
```

**Пояснення:** команда показує загальний обсяг оперативної пам'яті, скільки з неї зайнято, скільки вільно, а також обсяг та стан swap-простору.

---

## Короткий перелік команд (без виводу)

```bash
ps aux
top
ps aux --sort=-%mem | head -n 6
echo $$
ps -p $$ -o pid,ppid,comm,args

sleep 1000 &
jobs -l
fg %1
# Ctrl + Z
kill -9 <PID>
nohup sleep 1000 > nohup_sleep.log 2>&1 &

nice -n 10 sleep 1000 &
renice 15 -p <PID>
ulimit -a

df -h
free -h
```

---

## Висновок

У ході виконання роботи відпрацьовано навички перегляду й моніторингу процесів у Linux (`ps`, `top`), керування завданнями у фоновому та передньому режимах (`&`, `jobs`, `fg`, `kill`, `nohup`), налаштування пріоритетів виконання (`nice`, `renice`), перегляду обмежень ресурсів користувача (`ulimit`), а також моніторингу використання диска й оперативної пам'яті (`df`, `free`).


