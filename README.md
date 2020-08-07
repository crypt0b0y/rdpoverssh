# Приложение для работы с удаленным рабочим столом пк локальной сети через ssh (SSH + Kerio Control + RDP).
###### Принцип работы построен на функционале ssh. Интересный материал на эту тему: https://habr.com/ru/post/331348/#t5

!!!Внимание

1. Не используйте ключи авторизации ssh из выложеные в проекте. они исключительно для ознакомления. Мануалов по генерации ключа в сети полно, например тут https://habr.com/ru/post/122445/

2.текущая реализация предполагает наличие SSH сервера, доступного из сети Интернет, Kerio Control с авторизацией пользователей через MAC/IP адрес.

pick

В проекте используются библиотеки sshtunnel, PyQt5, threading и api Kerio Control доступный по ссылке https://manuals.gfi.com/en/kerio/api/control/reference/index.html или в качестве готовых python библиотек: **pykerio, python-kerio-api**

Подготовка серверной части:
Сервер SSH с белым IP. Для безопастности с ограниченным пользователем с параметром nologin или песочным bash, авторизацией по паролю или ключу.
Создать пользователя Kerio Control с ограниченными правами, только для чтения.
Kerio в качестве сервер шлюза с настройкой пользователей MS Active Directory и привязкой по MAC или IP для каждого пользователя.
Перенаправить внешний порт 3232 на порт сервера ssh.

#### Принцип работы:

Приложение подключается к SSH серверу, который находится за файерволом с проброшенным заранее портом, пробрасывает порт 4081 Kerio Contorol. 
Подключается к api для поиска ip по учетным данным поля "Фамилия" приложения.
Поиск ip происходит среди активных пк локальной сети.
Далее приложение отключается от api kerio и закрывает SSH соединение.  
Если ip не найден происходит выход. Если ip найден происходит повторное подключение к ssh серверу, но проброс порта происходит на найденный ip и порт RDP 3389.
Далее учетные данные вносятся в диспетчер учетных записей активного пользователя Windows.
Создается новый процесс с вызовом mstsc и передачей параметров подключения.
После успешного подключения учетные данные удаляются.



#### Настройка перед компиляцией в exe:

Основной файл запуска программы remotecli.py
Файл настроек подключений sshconnect.py
Файл дизайна для python desing.py
Файл графических ресурсов res.py

Основной файл настройки подключения sshconnect.py
```
publicipadress = ('REMOTE_PUBLIC_IP', 3232) # настройка ip адреса подключения к ssh серверу.
def sshtunconnect (address):
    ssh_username="LOGIN_SSH", # логин для подключения к ssh серверу 
    ssh_password="PASSWORD_SSH", # пароль для подключения к ssh серверу
    ssh_pkey="srv.key", # файл ключа для подключения к ssh серверу
    local_bind_address=('localhost', 2222) #  адрес и порт откуда происходит проброс

def sshtungetip():
    remote_bind_address=('IP_Kerio_FIREWALL', 4081), # ip адрес и порт kerio control
   
def rdpdataconnection():
    subprocess.call(f"cmdkey /add:localhost /user:DOMAINMAIN\{login} /pass:{password}") # Если пользователи не доменные DOMAIN\ убрать  
```
Файл kerio/kerio.py
```
username = "kerio_admin_read_access_login" логин от kerio control, достаточно ограниченной учетной записи с правами только для чтения
password = "kerio_admin_read_admin_pass"    пароль от kerio control
```


Планы:

:black_square_button: Делать или не делать вот в чем вопрос?
    
1. Реализация раздела настройки.
    
2. Реализация ручного указания ip на в настройках приложения.

3. Реализация получения ip из БД, с предварительно занесенных туда пользователей (клиент - серверная версия) через веб интерфейс.
    
....
