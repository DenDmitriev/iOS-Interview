# Модель OSI для начинающих

![модель оси](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/32fbbd5b-b8e0-489a-9dfa-c4d86bcd9eea)

<table>
    <tr>
        <td>Уровень</td>
        <td>Картинка</td>
        <td>Тип данных</td>
        <td>Функции</td>
        <td>Описание</td>
        <td>Пример</td>
        <td>Оборудование</td>
    </tr>
    <tr>
        <td>Прикладной (application)</td>
        <td><img width="260" alt="" src=""></td>
        <td>Данные</td>
        <td>Доступ к сетевым службам</td>
        <td>Уровень взаимодействия человека и компьютера, где приложениям доступны сетевые службы</td>
        <td>HTTP, FTP, POP3, SMTP, WebSocket</td>
        <td>Хосты (клиенты сети), Межсетевой экран</td>
    </tr>
    <tr>
        <td>Представления (presentation)</td>
        <td><img width="260" alt="" src=""></td>
        <td>Данные</td>
        <td>Представление и шифрование данных</td>
        <td>Уровень, где гарантируется использование в нужном формате, осуществляется шифрование</td>
        <td>Кодировки ASCII, EBCDIC, SSL, gzip</td>
        <td>null</td>
    </tr>
    <tr>
        <td>Сеансовый (session)</td>
        <td><img width="260" alt="" src=""></td>
        <td>Данные</td>
        <td>Управление сеансом связи</td>
        <td>Уровень, где поддерживается сеанс связи, надежное соединение двух сторон</td>
        <td>RPC, PAP, L2TP, gRPC</td>
        <td>null</td>
    </tr>
    <tr>
        <td>Транспортный (transport)</td>
        <td><img width="260" alt="TCP" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/523dd5dc-f154-4184-a249-8d399ed3ae14"></td>
        <td>Сегменты/Датаграммы проще - блоки</td>
        <td>Прямая связь между конечными пунктами и надёжность</td>
        <td>Уровень, где данные передаются с помощью протоколов ТСР и UDP</td>
        <td>TCP, UDP, SCTP, Порты</td>
        <td>null</td>
    </tr>
    <tr>
        <td>Сетевой (network)</td>
        <td><img width="260" alt="map" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/f9d05295-9751-4654-abd3-24793d6643b5"></td>
        <td>Пакеты (packet)</td>
        <td>Определение маршрута, IP и логическая адресация</td>
        <td>Уровень, где осуществляется маршрутизация</td>
        <td>IPv4, IPv6, IPsec, AppleTalk, ICMP</td>
        <td>"Маршрутизатор, Сетевой шлюз,</td>
    </tr>
    <tr>
        <td>Канальный (data link)</td>
        <td><img width="260" alt="Router" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/a4ed641c-fff7-4d8f-864b-f735d5f1bb1d"></td>
        <td>Биты (bit)/Кадры (frame)</td>
        <td>Физическая адресация МАС и LLC</td>
        <td>Уровень, где формируются пакеты</td>
        <td>PPP, IEEE 802.22, Ethernet, DSL, ARP, сетевая карта.</td>
        <td>Межсетевой экран"</td>
    </tr>
    <tr>
        <td>Физический (physical)</td>
        <td><img width="260" alt="Концентратор" src="https://github.com/DenDmitriev/iOS-Interview/assets/65191747/b100cbdd-9043-46dc-9fff-4b2550eb0ae7"></td>
        <td>Биты (bit)</td>
        <td>Работа со средой передачи (кабель, безпроводная), сигналами и двоичными данными</td>
        <td>Уровень, где передаются электрических сигналы по физическому каналу связи</td>
        <td>USB, RJ («витая пара», коаксиальный, оптоволоконный), радиоканал</td>
        <td>"Сетевой мост, Коммутатор,</td>
    </tr>
</table>
![Unknown-4](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/523dd5dc-f154-4184-a249-8d399ed3ae14)

![Network-Diagram-Example-3](https://github.com/DenDmitriev/iOS-Interview/assets/65191747/f9d05295-9751-4654-abd3-24793d6643b5)
