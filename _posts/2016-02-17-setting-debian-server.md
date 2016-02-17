---
title: "Первичная настройка debian 8 сервера"
tags: [debian, linux]
---
После установки debian на сервере необходимо произвести первичную настройку. В первую очередь коннектимся к серверу через ssh. IP адрес указываем тот, который был выдан хостером:
{% highlight bash %}
ssh root@123.123.123.123
{% endhighlight %}
После выхода дистрибутива наверняка некоторые пакеты были обновлены. А мы понимаем, что желательно, чтобы система была актуальна и все обновления стояли. Поэтому, обновляем индекс пакетов и сами пакеты.
{% highlight bash %}
apt-get update && apt-get upgrade
{% endhighlight %}
Настраиваем время на сервере.
{% highlight bash %}
dpkg-reconfigure tzdata
{% endhighlight %}
Устанавливаем локаль.
{% highlight bash %}
dpkg-reconfigure locales
{% endhighlight %}
Работать под рутом не комильфо, поэтому заводим пользователя для работы. Все команды, требующие административных прав будем вводить через sudo.
{% highlight bash %}
adduser deploy
adduser deploy sudo
exit
ssh deploy@123.123.123.123
{% endhighlight %}

Теперь необходимо настроить доступ по ssh. 
Создаем директорию ssh и меняем права на доступ к данной директории. 700 в данном случае означает, что доступ будет иметь только владелец.
{% highlight bash %}
mkdir .ssh
chmod 700 .ssh
{% endhighlight %}
После этого генерируем на локальном ПК ключ, если он еще не сгенерирован.
{% highlight bash %}
ssh-keygen -t rsa
{% endhighlight %}
Копируем его на сервер
{% highlight bash %}
scp ~/.ssh/id_rsa.pub deploy@123.123.123.123:~/.ssh/authorized_keys
{% endhighlight %}
Запрещаем вход под рутом через ssh и вход по паролю. Для этого редактируем конфиг ssh.
Открываем sshd_config в редакторе nano
{% highlight bash %}
sudo nano /etc/ssh/sshd_config
{% endhighlight %}
Находим в нем данные параметры и меняем yes на no
{% highlight bash %}
PermitRootLogin no
PasswordAuthentication no
{% endhighlight %}
После чего перезапускаем службу
{% highlight bash %}
sudo systemctl restart sshd
{% endhighlight %}
Смотрим запущенные сетевые службы
{% highlight bash %}
sudo netstat -tulpn
{% endhighlight %}
Убираем ненужное из автозагрузки (мы же настраиваем веб сервер)
{% highlight bash %}
update-rc.d rpcbind disable
update-rc.d exim4 disable
{% endhighlight %}
Настраиваем iptables
{% highlight bash %}
sudo nano /tmp/v4
{% endhighlight %}
{% highlight bash %}
*filter

# Allow all loopback (lo0) traffic and reject traffic
# to localhost that does not originate from lo0.
-A INPUT -i lo -j ACCEPT
-A INPUT ! -i lo -s 127.0.0.0/8 -j REJECT

# Allow ping.
-A INPUT -p icmp -m state --state NEW --icmp-type 8 -j ACCEPT

# Allow SSH connections.
-A INPUT -p tcp --dport 22 -m state --state NEW -j ACCEPT

# Allow HTTP and HTTPS connections from anywhere
# (the normal ports for web servers).
-A INPUT -p tcp --dport 80 -m state --state NEW -j ACCEPT
-A INPUT -p tcp --dport 443 -m state --state NEW -j ACCEPT

# Allow inbound traffic from established connections.
# This includes ICMP error returns.
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Log what was incoming but denied (optional but useful).
-A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables_INPUT_denied: " --log-level 7

# Reject all other inbound.
-A INPUT -j REJECT

# Log any traffic which was sent to you
# for forwarding (optional but useful).
-A FORWARD -m limit --limit 5/min -j LOG --log-prefix "iptables_FORWARD_denied: " --log-level 7

# Reject all traffic forwarding.
-A FORWARD -j REJECT

COMMIT
{% endhighlight %}
{% highlight bash %}
sudo nano /tmp/v6
{% endhighlight %}
{% highlight bash %}
*filter

# Allow all loopback (lo0) traffic and reject traffic
# to localhost that does not originate from lo0.
-A INPUT -i lo -j ACCEPT
-A INPUT ! -i lo -s ::1/128 -j REJECT

# Allow ICMP
-A INPUT -p icmpv6 -j ACCEPT

# Allow HTTP and HTTPS connections from anywhere
# (the normal ports for web servers).
-A INPUT -p tcp --dport 80 -m state --state NEW -j ACCEPT
-A INPUT -p tcp --dport 443 -m state --state NEW -j ACCEPT

# Allow inbound traffic from established connections.
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Log what was incoming but denied (optional but useful).
-A INPUT -m limit --limit 5/min -j LOG --log-prefix "ip6tables_INPUT_denied: " --log-level 7

# Reject all other inbound.
-A INPUT -j REJECT

# Log any traffic which was sent to you
# for forwarding (optional but useful).
-A FORWARD -m limit --limit 5/min -j LOG --log-prefix "ip6tables_FORWARD_denied: " --log-level 7

# Reject all traffic forwarding.
-A FORWARD -j REJECT

COMMIT
{% endhighlight %}
Применяем данные правила
{% highlight bash %}
sudo iptables-restore < /tmp/v4
sudo ip6tables-restore < /tmp/v6
{% endhighlight %}
Устанавливаем iptables-persistent
{% highlight bash %}
sudo apt-get install iptables-persistent
{% endhighlight %}
Отвечаем Yes на оба запроса сохранить текущие правила, и удаляем временные файлы
{% highlight bash %}
sudo rm /tmp/{v4,v6}
{% endhighlight %}
