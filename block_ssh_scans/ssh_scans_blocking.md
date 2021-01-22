### Блокировка сканов порта ssh
Блокировка делается на основе `iptables`+`ipset` довольно простым скриптом
, запускаемым из crontab пользователя `root`.

Принцип работы. 

* Раз в 2 минуты по крону запускается скрипт, который находит
  в `/var/log/auth.log` записи с неудачными попытками входа
  (нужно больше 2-х неудачных попыток входа).
* Соответствующие IP проверяются, что они еще не заблокированы,
* если нет - заливается глобальная база IP, которые нужно заблокировать
* и, если найденного IP нет и там, то он блокируется отдельно (на сутки).

Скрипты лежат в `/root/bin` запускаются по крону
```
*/2 * * * *  /root/bin/block_scans.sh
```
Настройки iptables:
```
*filter
:INPUT ACCEPT [0:0]
:IN_SSH - [0:0]
-A INPUT -p tcp -m tcp --dport 22 -j IN_SSH
-A IN_SSH -m set --match-set ssh_scans src -j DROP
-A IN_SSH -m set --match-set ssh_blocklist src -j DROP
-A IN_SSH -j ACCEPT
COMMIT
```
Поиск IP идет по специализированной базе (ipset), т.к. эта проверка делается
для каждого IP пакета, пришедшего на порт 22. Настройки ipset:
```
create ssh_scans hash:ip family inet hashsize 1024 maxelem 65536 timeout 86400 
create ssh_blocklist hash:ip family inet hashsize 1024 maxelem 65536 timeout 86400
```
Для `nftables` у меня есть аналогичный скрипт, но в Ubuntu на него еще не перешли.