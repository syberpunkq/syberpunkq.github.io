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
